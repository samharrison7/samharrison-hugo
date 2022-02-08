---
title: "Some Content"
date: 2021-12-23T12:18:46Z
tags: ['hello', 'nanofase', 'speed']
categories: ['A category']
draft: false
---

This is a post!

```python
import pandas as pd

df = pd.read_csv('/path/to/file.csv')
print(df)
print("Some text")

def times_two(stuff):
    return stuff * 2
```

```fortran
program main
    implicit none
    character(len=256) :: my_char = "This is a character string"
    print *, my_char

    function times_two(stuff) result(rslt)
        real(dp), intent(in) :: stuff
        real(dp), intent(out) :: rslt
        rslt = stuff * 2
    end function
end program
```

```html
<div class="my-class">
    <span id="spanny">Stuff</span>
</div>
```