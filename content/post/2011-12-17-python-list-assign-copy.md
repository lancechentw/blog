+++
title = "Python: list assigning(copying)"
date = "2011-12-17T11:49:00+08:00"
type = "post"
tags = ["Python"]
+++

```pycon
>>> a = [1,2,3,4,5]
>>> b = a
>>> b[0] = 10
>>> print a
[10,2,3,4,5]
```

在python中，list的assign有點類似c的pointer，改變b的值同時也會改變a的值，要作到
真正的copy有兩種方法：

### slicing
```pycon
>>> b = a[:]
```
這個作法雖然簡單，但是在nested的結構中會有問題，可以改採用第二種方法

### module copy
```pycon
>>> import copy
>>> b = copy.copy(a)
```
    
module copy中又有copy與deepcopy，兩者的差別要再研究研究
  
Reference:  
[python module copy](http://docs.python.org/library/copy.html)

