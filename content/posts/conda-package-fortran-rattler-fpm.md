+++ 
draft = true
date = 2024-11-02T20:11:48Z
title = "Creating a Conda package from Fortran using fpm and rattler-build"
description = "How to create a Conda package from Fortran code using the Fortran Package Manager (fpm) and rattler-build"
slug = "conda-package-fortran-rattler-fpm"
tags = ["conda", "fortran", "fpm", "rattler-build", "pixi"]
+++

A while ago, I wrote a [post on how to create a Conda package from Fortran and Python code]({{< relref "conda-package-fortran-python" >}}). Though the main point of the post was to explore including Fortran code in Python, I also delved into Conda build as a way of building Conda packages. There is now a new kid on the block - [rattler-build](https://prefix-dev.github.io/rattler-build/latest/) - which offers a much faster and cleaner build experience for building Conda packages. I thought it was time for a blog post on that.

At the same time, the [Fortran Package Manager (fpm)](https://fpm.fortran-lang.org/) is maturing and becoming the de facto way of managing Fortran projects and packages. **Using fpm alongside rattler-build is a streamlined way of easily creating a Conda package for Fortran code**, demonstrating the power that this modern tooling offers to what is often (incorrectly) perceived as a legacy language.

And seeing as we're diving into rattler-build, which is made by [Prefix](https://prefix.dev), the lovely folk who also develop the [Pixi package manager](https://pixi.sh/latest/), it seems entirely appropriate to use Pixi as we go through this demo too.

{{< notice info "Pre-requisites" >}}
There is a fair bit of assumed knowledge in this article. In particular, you should be familiar with Conda packages and general principles of package and environment management. A bit of experience writing and building Fortran would also help.
{{< /notice >}}

## How it all fits together (aka, why two package managers?)

You might have noticed that I mentioned two package managers above - fpm and Pixi. Fpm is a package manager to deal with pure Fortran projects, and can do neat things like pull in external Fortran packages by providing (GitHub) links to their source code. Pixi, on the other hand, is completely language agnostic, like the Conda packages that it can install. In this demo, I use fpm to setup a project with the Fortran source code. You could easily use another Fortran build tool or a standalone Fortran compiler to do this, but hopefully by the end of the post, you will see the simplicity that fpm brings. Pixi is used to test out the installation of the Conda package that we create, as well as installing rattler-build and fpm. So, in summary:
- Fpm is used to setup and build the Fortran project, including installing an external Fortran library
- rattler-build is used to build the Fortran project into a Conda package by calling fpm as the build script
- Pixi is used to test out the package, and install rattler-build and fpm.

This is perhaps a bit over-the-top for the toy project here, but it sets the stage for how you might want to manage larger Fortran or multilingual projects.

{{< notice warning "rattler-build in Pixi?">}}
Prefix intend to integrate rattler-build into Pixi via a `pixi build` command. By the time you read this article, that might have already happened, so check out the [Pixi docs](https://pixi.sh) to see!
{{< /notice >}}

## Getting set up

Firstly, head over to [Pixi's docs](https://pixi.sh/latest/#installation) to install Pixi. Now, use Pixi to install ratter-build globally:

{{< command >}}
pixi global install rattler-build
{{< /command >}}

## Creating the Fortran project

It's just turned November as I write this, so what better project to create than a command line script that tells me how many days it is until Christmas?!

To do this, we will install fpm by creating a Pixi project in which to do so:

{{< command >}}
pixi init days-until-xmas 
cd days-until-xmas
pixi add fpm
{{< /command >}}

Note that by default, Pixi tries to find the package in the conda-forge channel. Neat! To check that fpm has been installed correctly, we can check its version, either by using `pixi run` to run the `fpm` command in this environment - `pixi run fpm --version` - or by activating the environment and starting a new shell there...

{{< command >}}
pixi shell
{{< /command >}}

...and then running fpm:

{{< command custom="(days-until-xmas) $" >}}
fpm --version
{{< /command >}}

We also need a Fortran compiler, and if you don't already have one installed, you can install GFortran from conda-forge using Pixi by running `pixi add gfortran` (if you have activated the environment by starting a new shell, `exit` this first).

Note that fpm also requires Git to be installed. Again, this can be installed at the same time as installing fpm by running `pixi add git`.

Now we can use fpm to create the Fortran project. By default, fpm (like Pixi) creates a new directory when you create a new project. You can disable that by using the `--backfill` command, but to do so, we must first step outside of the `days-until-xmas` directory:

{{< command custom="(days-until-xmas) $" >}}
cd ..
fpm new days-until-xmas --backfill
cd days-until-xmas
{{< /command >}}

Fpm creates an `fpm.toml` file and a few other default files, so alongside the `pixi.toml` and `pixi.lock` files that Pixi creates, your `days-until-xmas` directory should now look like this:

```
.
├── app
│   └── main.f90
├── fpm.toml
├── pixi.lock
├── pixi.toml
├── README.md
├── src
│   └── days-until-xmas.f90
└── test
    └── check.f90
```

Have a look in the `fpm.toml` to get a feeling of the metadata that fpm requires, and check out the [fpm manifest reference](https://fpm.fortran-lang.org/spec/manifest.html) for full details of what information can be provided via this file.

## Writing our app

If you look inside the `src/days-until-xmas.f90` and `app/main.f90` files, you will see a simple "Hello world"-type app that fpm has created by default. We can test this out to see how fpm works:

{{< command custom="(days-until-xmas) $">}}
fpm run
{{< /command >}}

You should see (after some compilation output) "Hello, days-until-xmas!" printed to your console. During this command, fpm has built the `src/days-until-xmas.f90` file into a static library `libdays-until-xmas.a`, compiled an executable from `main.f90` that calls this library, and then run the executable. This is the beauty of fpm: abstracting away the complexities of building and running Fortran code using sensible defaults and simple commands.

It gets even better if we want to use external libraries. Here, we want to calculate the number of days until Christmas, so let's use an external Fortran package, [datetime-fortran](https://github.com/wavebitscientific/datetime-fortran), to make it easier. This package is fpm-compatible (it has an `fpm.toml` file), meaning that we can easily specify that our project depends on it by adding the following to the bottom of our `fpm.toml` file:

```toml
[dependencies]
datetime.git = "https://github.com/wavebitscientific/datetime-fortran"
```

Now, if we run `fpm run` again, fpm also retrieves datetime-fortran from GitHub and compiles it into a static library, ready for use in our app.

You might have noticed the presence of a new directory, `build`. This is created by fpm and is where the compiled libraries and executables are stored. Have a look inside for a feeling of how fpm works. If you're intending to version control your project via Git, you will probably want to add this `build` directory to the `.gitignore` file (that Pixi has already added a couple of things to).

Now let's write our app, using the datetime-fortran library. In `src/days-until-xmas.f90`, we write:

```fortran
module days_until_xmas
  use datetime_module, only: datetime, timedelta
  implicit none
  private

  public :: get_days_until_xmas

contains

  function get_days_until_xmas()
    !! Function that returns the number of days until Christmas
    !! as an integer
    type(datetime) :: current_date
    type(datetime) :: xmas
    type(timedelta) :: date_delta
    integer :: get_days_until_xmas

    ! Get the current date
    current_date = current_date%now()
    ! Get Christmas day this year
    xmas = datetime(current_date%getYear(), 12, 25)
    ! Difference between now and Christmas
    date_delta = xmas - current_date
    ! Get the number of days from this difference
    get_days_until_xmas = date_delta%getDays()
  end function

end module days_until_xmas
```

In `app/main.f90`, we write:

```fortran
program main
  use days_until_xmas, only: get_days_until_xmas
  implicit none

  print *, 'Number of days until Christmas:', get_days_until_xmas()
end program main
```

Now we can test that our program functions created. It is 2nd November as I write this, and so there should be 52 days until Christmas:

{{< command command="fpm run" custom="(days-until-xmas) $" >}}
days-until-xmas.f90                    done.
libdays-until-xmas.a                   done.
main.f90                               done.
days-until-xmas                        done.
[100%] Project compiled successfully.
 Number of days until Christmas:          52
{{< /command >}}

Our app is complete! Now we need to write a recipe so that rattler-build knows how to build it.

## Writing a rattler-build recipe

Rattler-build works in a similar way to conda-build: You specify a `recipe.yaml` file, and rattler-build builds a package using the instructions provided in that. The `recipe.yaml` file is similar, but not identitical to, the conda-forge schema. The [Prefix docs](https://prefix-dev.github.io/rattler-build/latest/highlevel/) give a good intro.

It's up to you where you place your recipe, but it can't be in the same directory as the directory the build outputs to, which by default is `output`. A common practice is to have a separate `recipe` folder to place your recipe in. As we're done interacting with fpm for the moment (and therefore the Pixi environment), you can `exit` out of the Pixi shell.

{{< command >}}
mkdir recipe
touch recipe/recipe.yaml
{{< /command >}}

Use whatever editor you like to add the following recipe to that file. We'll go through the contents in a moment.

```yaml
package:
  name: days-until-xmas
  version: 1.0.0

source:
  path: ..

build:
  script: fpm install --prefix=${PREFIX}

requirements:
  build:
    - ${{ compiler('fortran') }}
    - ${{ compiler ('c') }}
    - ${{ compiler ('cxx') }}
    - git
    - fpm
```

The `package` section provides useful metadata about our package. The `source` section tells rattler-build where our package is (we could specify a URL here instead). The `build` section tells rattler-build how to build the package. This is where we link up rattler-build with fpm, by using fpm within our build script to compile the Fortran code. We tell fpm to install this compiled code in `${PREFIX}`, which is the root directory of our built package.

The `requirements` section is where we tell rattler-build what packages we need to build and run the package. The recommended way of specifying compilers is to use the `${{ compiler('lang') }}` template function. This takes care of figuring out the runtime dependencies instead of having to manually add libraries like `libgfortran` to the `host` section of `requirements`. Because of this, we only need to specify requirements in the `build` section. We need a Fortran, C and C++ compiler, the latter two for the datetime-fortran library. By default, rattler-build will use GNU compilers for these. See the [rattler-build docs](https://prefix-dev.github.io/rattler-build/latest/compilers/) for info on how to control which compiler and version is used.

{{< notice tip "Tests" >}}
It is good practice to write tests! We've glossed over this here, but really we should have written a few tests in the Fortran source code (fpm has functionality for running tests) and specified that they be run when building of package via the `test` section (which could be as simple as running `fpm test`).
{{< /notice >}}

## Building and installing the package

We can now use rattler-build to build the package:

{{< command >}}
rattler-build build --recipe recipe/recipe.yaml
{{< /command >}}

Those of you that are used to conda-build will be astonished at how fast rattler-build is!

Rattler-build places the built package by default in the `output` directory. The full path to the package is given somewhere near the end of the build output. Copy this to the clipboard so that we can test installing it.

We can test the installation using anything that installs Conda packages, such as Conda, Mamba or Pixi. Seeing as we've been using Pixi here, let's create a test Pixi project to install our package into:

{{< command >}}
cd ..
pixi init test-days-until-xmas
cd test-days-until-xmas
{{< /command >}}

Now we can install our package by calling `pixi add` on the path to the package:

{{< command >}}
pixi add /path/to/package/output/linux-64/days-until-xmas-1.0.0-hb0f4dca_0.conda
{{< /command >}}

Change the `/path/to/package...` path to that which you just copied to the clipboard. Providing the package installed correctly, we can now test it by running the `days-until-xmas` command:

{{< command command="pixi run days-until-xmas" >}}
 Number of days until Christmas:          52
{{< /command >}}

## Next steps

You now have a shiny new functioning Conda package! Naturally, the next logical step would be to upload this to an online repository like Anaconda. We won't do this in this article, other than to mention that rattler-build has a very useful [`rattler-build upload` command](https://prefix-dev.github.io/rattler-build/latest/authentication_and_upload/#uploading-packages) to take care of this task.


## Final words

I hope that you have found this post useful! I have placed a copy of the scripts that I created in this post on Codeberg*, so if you simply want to test the steps above without creating your own app, feel free to clone this and have a play around:

{{< button href="https://codeberg.org/samharrison7/days-until-xmas" icon="code" text="Get the code" target="_blank" >}}

*Like GitHub, but open-source - because why should you host open-source software on a closed-source service?