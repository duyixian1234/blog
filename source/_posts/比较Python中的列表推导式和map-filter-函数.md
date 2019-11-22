---
title: '比较Python中的列表推导式和map(),filter()函数'
date: 2019-11-22 10:21:17
tags:
---

# 比较Python中的列表推导式和map(),reduce()函数

对一个列表（迭代器）中的元素进行批量处理是一个很常见的业务需求，在Python中，一般有三种解决方案：`for`循环，列表推导式，或者`map()`,`filter()`函数。

例如我们计算一下100以内奇数的平方和。

```python
# for loop
total = 0
for x in range(100):
    if x % 2:
        total += x * x
```
```python
# list comprehension
sum(x * x for x in range(100) if x % 2)
```

```python
# map(), filter()
sum(map(lambda x: x * x, filter(lambda x: x % 2, range(100))))
```

`for`循环方案最容易理解但有些繁琐，列表推导式方案就简洁了很多，`map()`,`filter()`方案存在一个问题就是要理解它们嵌套关系和执行顺序。

三种方案的效率也可以进行一下比较。

![](/image/0001.png)

可以看到for循环和列表推导式的效率是相近的，而`map()`,`filter()`方案就慢很多，这是因为`map()`,`filter()`方案中进行了大量的函数调用，而Python解释器对列表推导式有专门的优化。

我按照自己的尺度给三种方案做了一个评价。


|方案  |可理解度  |简洁度  | 执行效率 |
|---------|---------|---------|---------|
|for loop     |   ★★★      |   ★      |★★★|
|list comprehension     |   ★★      |   ★★★      |★★★|
|map(),filter()     |  ★★       |  ★★       |★|

综合而言，在Python中进行列表（迭代器）的处理，列表推导式是更简洁，效率更高的方案，也更Pythonic，不过当列表推导式过于复杂的时候，转而使用`for`循环会使代码更好理解和可维护。


