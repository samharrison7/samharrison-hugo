+++ 
draft = false
date = 2024-07-02T10:07:12+01:00
title = "Scheduling a Python script using PythonAnywhere and cron-job.org"
description = "A walk-through of using PythonAnywhere and cron-job.org to schedule a Python script"
tags = ["python", "cron"]
images = []
+++

Scheduling a script is one of those jobs that sounds like it should be easy, but is surprisingly tricky. Here, I walk you through one of the easier ways of doing so, by hosting the script on [PythonAnywhere](https://www.pythonanywhere.com/) to give it a URL, and scheduling it to run using [cron-job.org](https://cron-job.org), both of which offer free services.

## What's so difficult?

Scheduling scripts is when, instead of manually running a script whenever you need to, you tell a scheduler to run it at a given frequency or based on other events. There are many uses, such as running a forecasting model to provide regular forecasts, or pulling in the latest observations data to parameterise that model.

You could do this manually, but that's just a pain. You could write scheduling into your script with liberal use of `sleep()`, but that relies on your script constantly running, which is hardly ideal. Schedulers like [`cron`](https://en.wikipedia.org/wiki/Cron) on Linux give you the ability to automate the running of scripts at given frequencies, so why can't you just set that going on your computer? You could, but that relies on your computer being on when you want the script to run, which again, is hardly ideal.

There are countless options for getting around this, but most revolve around hosting the script on a server instead. That could be a local server, for example if you have a Raspberry Pi or similar, or a cloud server. For this example, we will use the excellent [PythonAnywhere](https://www.pythonanywhere.com/), which provides a very straightforward (and free) way of hosting Python apps. We will use [Flask](https://flask.palletsprojects.com/en/3.0.x/) to give a URL to our script (that is, a URL that, when someone accesses it, the script runs).

Hosting your script is only one part of the solution though - you still need a scheduler to run the script. If you have sufficient access to your server, you could of course set a `cron` job going on it. In fact, PythonAnywhere offers a limited scheduling service via its interface. A more general (and arguably robust) solution is to use a separate scheduler, such as the free [cron-job.org](https://cron-job.org), and that is what we explore here.

So, the simple "I just need to schedule this script" has turned into:
* Write the script
* Write a web app around the script to enable it to be run (using Flask)
* Host that app on a cloud server (PythonAnywhere)
* Use a scheduler (cron-job.org) to issue a request to your app's URL at a given frequency

{{< notice tip "Requirements" >}}
All you need installed to run through this tutorial is a relatively recent version of Python and the Flask library. To set up a Conda/Mamba environment called `schedule` that uses these, simply run `mamba create -n schedule -c conda-forge python flask`.
{{< /notice >}}

## The script

For demonstrative purposes, let's keep it simple: We want to get the current temperature at a given location, using the excellent open-source weather API [Open-Meteo](https://open-meteo.com/), and save that to a local file, which we call `meteo.py`:

```python
import requests
from datetime import datetime

def log_current_temperature():
    """
    Log the current temperature at the specified location
    to a data file.
    """

    # What location are we interested in?
    lat, lon = 54.05, -2.80

    # Get the current surface temperature at that location
    # using Open-Meteo
    r = requests.get(f'https://api.open-meteo.com/v1/forecast?latitude={lat}&longitude={lon}&current=temperature_2m')

    # If successful, append to the data file
    if r.status_code == 200:
        # Get the temperature from the returned data
        temperature = r.json()['current']['temperature_2m']
        # Write this and the current time to the file
        with open('data.csv', 'a') as f:
            f.write(f'{datetime.now().strftime("%Y-%m-%d %H:%M:%S")},{temperature}\n')
```

This file defines a function `log_current_temperature()`, which sends a request to the Open-Meteo API, gets the current temperature from the response, and appends a CSV file with the current timestamp and this temperature.

## The Flask app

Because we are going to be running this script on a server, we need something to tell the server to run our function when the user goes to a particular URL. [Flask](https://flask.palletsprojects.com/en/3.0.x/) is a "micro web framework" that makes this incredibly easy, so we will use that here. All we need to do is write an application file, `app.py`, that defines our endpoint (the URI that runs `log_current_temperature()`):

```python
from flask import Flask
from meteo import log_current_temperature

# Create the Flask app
app = Flask(__name__)

# Define the endpoint to log the current temperature
@app.route('/log')
def log_temperature():
    # Call the function to log the temperature
    log_current_temperature()
    # We return an empty string because we don't actually
    # need to return anything to the user
    return ''

if __name__ == '__main__':
    app.run()
```

We can test our Flask app locally:

```shell script
$ flask run
```

This will start Flask as a local server, usually at port 5000. In the app, we defined our endpoint as `/log`, meaning you that you can trigger the current temperature to be logged by going to https://localhost:5000/log.

{{< notice tip >}}
If you don't have admin permissions and get a message along the lines of "An attempt was made to access a socket in a way forbidden by its access permissions", then try a different port number by using the `-p` flag - sometimes lower port numbers are reserved for admin only. For example, try `flask run -p 8000`.
{{< /notice >}}

Your app directory should now look like this, with the newly created `data.csv` file containing a record for the current temperature:

```
.
├── app.py
├── data.csv
└── meteo.py
```

## Hosting the app

We will host the app on [PythonAnywhere](https://www.pythonanywhere.com/), which makes hosting Flask apps very straightforward. Head to the website and create yourself an account. We then need to upload our scripts to the server, which can either be done via the online interface or programmatically via a Bash console, which again can be opened via the online interface (paid accounts also get SSH access).

Place your files in a new directory on the server, which we'll call `openmeteo-demo`. We then need to tell PythonAnywhere about our app, which can be done via the "Web" tab, at the URL `https://www.pythonanywhere.com/user/<your-username>/webapps/`. Click the "+ Add a new web app" button and (if you have a free account), acknowledge the notice that you are not allowed a custom domain. When prompted, choose to create a Flask app and choose a Python version - I tested Python 3.10 with this demo, so that is a safe bet. When prompted to enter the path for a new Flask project, leave it as default - if we specify the path to our app, it will be overwritten.

{{< notice tip "Using a virtual environment?" >}}
Our app is simple enough to not rely on external packages (except for Flask), but if yours does, then you will want to set up a virtual environment to manage this. PythonAnywhere supports using `virtualenv`, and there are thorough [instructions on how to do so here](https://help.pythonanywhere.com/pages/Virtualenvs/).
{{< /notice >}}

On the dashboard page for the web app, scroll down to the "Code" section, which is where we tell PythonAnywhere where our app is. Change "Source code" to `/home/<username>/openmeteo-demo` (where `<username>` is your username), "Working directory" to the same, and then click on the "WSGI configuration file" to edit it - this is where we tell the server where exactly the Flask app is. We need to change the project home to `/home/<username>/openmeteo-demo`, and import our Flask app from the file `app.py`. Your WGSI config file should now look something like this:

```python
import sys

# add your project directory to the sys.path
project_home = '/home/<username>/openmeteo-demo'
if project_home not in sys.path:
    sys.path = [project_home] + sys.path

# import flask app but need to call it "application" for WSGI to work
from app import app as application  # noqa
```

At the top of the app dashboard, click the button to reload the app. We can now visit our app's URL to check that a new data file is created on the server. Your URL should be `https://<username>.pythonanywhere.com/log` - visit this and then check that the `data.csv` file has been created and has one entry with the current temperature. If all is good, you can delete this file ready to be re-created when you set up your cron job.

## Scheduling via cron-job.org

Now that we have a URL that runs our script, we need something to run it periodically. PythonAnywhere offers a very limited ability to run scripts on a daily basis on its free tier, but to give ourselves more flexibility, we will use [cron-job.org](https://cron-job.org). Head to the website, create yourself an account and click the button to "Create Cronjob". Give your job a name, then add the URL `https://<username>.pythonanywhere.com/log` and choose a schedule of your choice - bearing in mind that Open-Meteo is a free service who requests that you keep daily requests below 10,000 (roughly once every 10 seconds). Once you have created your job, you can view its status via the Conjobs tab.

And we're done! Of course, this example is arbitrary, and one big improvement that we could make would be to do something more meaningful with the temperature data, rather than just saving it to a local server file. If you want a challenge, try and use the Google Cloud API to save the data to a Google Spreadsheet instead of the local file.

## Final words

I hope that you have found this post useful! I have placed a copy of the scripts that I created in this post on Codeberg*, so if you simply want to test the steps above without creating your own app, feel free to clone this and have a play around:

{{< button href="https://codeberg.org/samharrison7/openmeteo-demo" icon="code" text="Get the code" target="_blank" >}}

*Like GitHub, but open-source - because why should you host open-source software on a closed-source service?