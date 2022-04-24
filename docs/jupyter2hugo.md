# Converting Jupyter Notebooks to Hugo Markdown files

This is mostly taken care of by the `nb2hugo` script:

```shell
$ nb2hugo notebooks/<name>.ipynb --site-dir <site-dir> --section posts
```

Plotly requres a little extra work though. Either set the renderer as a static image, e.g.

```python
fig.show(renderer='png')
```

Or, set the renderer as an iframe, and manually copy the generated iframe files (`nb2hugo` doesn't take care of this).

```python
fig.show(renderer='iframe')
```

You'll need to look in the generated `.md` file to find the name of the generated `.html` file to copy:

```shell
$ cp notebooks/iframe_figures/<figure_*.html> static/posts/<name>/
```