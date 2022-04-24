+++
title = "My notebook title"
date = "2017-06-01"
author = "firstname lastname"
categories = ["category0", "category2"]
tags = ["tag0", "tag2", "tag3"]
+++



```python
import numpy as np

arr = np.array([0,1,2,3,4])
arr
```




    array([0, 1, 2, 3, 4])



Here is an equation:

\\[
E = mc^2
\\]

where \\(E\\) is energy...


```python
import matplotlib.pyplot as plt

plt.plot(arr)
plt.show()
```


    
![png](output_3_0.png)
    



```python
import plotly.express as px

fig = px.scatter(arr)
fig.show(renderer='iframe')
```


{{< rawhtml >}}
<iframe
    scrolling="no"
    width="100%"
    height="545px"
    src="iframe_figures/figure_14.html"
    frameborder="0"
    allowfullscreen
></iframe>

{{< /rawhtml >}}


Hello!
