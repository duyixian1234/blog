---
title: 初识Python3.7的dataclasses标准库
date: 2020-03-05 16:50:06
tags: python
categories: python
---

# 初识 Python3.7 的 dataclasses 标准库

最近在进行一个新的后端项目时想初步应用一下领域驱动设计的思想。DDD 开发需要对一个领域对象进行各种操作，而不是把业务数据包在`dict`里在 `action` 层,`repo` 层中传来传去。如何方便高效地定义实体类成为一个重要前提。Python 3.7 版本引入的新标准库 dataclasses 可以帮助我们解决这个问题。

## 1. dataclasses 的简介和使用

dataclasses 的官方介绍是：

> This module provides a decorator and functions for automatically adding generated special methods such as **init**() and **repr**() to user-defined classes. It was originally described in PEP 557.

使用 dataclasses 我们可以很方便地利用类型注解类定义数据类。

```python
from dataclasses import dataclass
@dataclass
class InventoryItem:
    '''Class for keeping track of an item in inventory.'''
    name: str
    unit_price: float
    quantity_on_hand: int = 0

    def total_cost(self) -> float:
        return self.unit_price * self.quantity_on_hand
```

使用了`dataclass`函数装饰器装饰`InventoryItem`类后，会生成`__init__`方法：

```python
def __init__(self, name: str, unit_price: float, quantity_on_hand: int=0):
    self.name = name
    self.unit_price = unit_price
    self.quantity_on_hand = quantity_on_hand
```

然后就可以直接实例化`InventoryItem`对象`InventoryItem('apple',1,10),InventoryItem('banana',1,quantity_on_hand=10)`

实际上`dataclass`函数是有很多参数的:

```python
@dataclasses.dataclass(*, init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)
```

这里介绍一下比较常用的参数：

- init:是否生成`__init__`方法，如果用户手动定义的`__init__`方法，这个属性会被忽略。
- repr:是否生成`__repr__`方法。
- eq:是否生成`__eq__`方法以用'=='比较不同实例，会按定义顺序比较对象中的字段。
- order:是否生成`lt`,`gt`等方法用于比较不同实例。
- frozen:默认为 `False`,设置为 `True` 的话对这个类实例的字段进行复制会抛出异常，相当于定义一个不可变对象

上文定义的`InventoryItem`类没有指定参数，全部是默认参数，具体的表现行为如下：

```python
In [1]: from dataclasses import dataclass
   ...: @dataclass
   ...: class InventoryItem:
   ...:     '''Class for keeping track of an item in inventory.'''
   ...:     name: str
   ...:     unit_price: float
   ...:     quantity_on_hand: int = 0
   ...:
   ...:     def total_cost(self) -> float:
   ...:         return self.unit_price * self.quantity_on_hand
   ...:

In [2]: a = InventoryItem('a',1,10) # 定义了__init__方法

In [3]: b = InventoryItem('b',2,5)

In [4]: a # 定义了__repr__方法
Out[4]: InventoryItem(name='a', unit_price=1, quantity_on_hand=10)

In [5]: a == b # 定义了__eq__方法
Out[5]: False

In [6]: a == InventoryItem('a',1,10)
Out[6]: True

In [7]: a < b # 没有定义'__lt__'f方法
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-7-c41276b34900> in <module>
----> 1 a < b # 没有定义'__lt__'方法

TypeError: '<' not supported between instances of 'InventoryItem' and 'InventoryItem'

In [8]: a.name = 'aa' # forzen 属性为False,可以修改字段

In [9]: a
Out[9]: InventoryItem(name='aa', unit_price=1, quantity_on_hand=10)
```

## 2. 使用`dataclasses.field`修饰字段

事实上我们不仅可以通过对 dataclass 的参数定义数据类的整体表现，也可以指定具体字段的行为，只需要用到`dataclasses.field`函数。

```python
dataclasses.field(*, default=MISSING, default_factory=MISSING,\
 repr=True, hash=None, init=True, compare=True, metadata=None)
```

介绍一下`field`函数的常用参数：

- default: 设置默认值
- default_factory: 设置默认工厂函数
- repr: 在`__repr__`方法中是否展示这个字段
- init: 在`__init__`方法中是否需要初始化这个字段

其中`default_factory`属性在一些情况下作用相当大。例如我们定义如下数据类：

```python
@dataclass
class A:
    nums:list = []
```

如果一切正常的话我们实例化两个对象`a`和`b`，然后`a.nums.append(1)`，`b.nums.append(2)`，此时`a.nums == b.nums == [1, 2]`,因为`a.num`和`b.num`实际指向的都是定义`A`时初始化过的那个空列表。

好在上述代码是无法运行的，会抛出异常`ValueError: mutable default <class 'list'> for field nums is not allowed: use default_factory`。按照提示，我们应该制定`nums`的默认工厂函数：

```python
In [1]: from dataclasses import dataclass,field

In [2]: @dataclass
   ...: class A:
   ...:     nums:list = field(default_factory=list)
   ...:

In [3]: a = A()

In [4]: b = A()

In [5]: a.nums.append(1)

In [6]: b.nums.append(2)

In [7]: a.nums, b.nums
Out[7]: ([1], [2])
```

我们指定了`nums`字段的默认工厂函数是`list`,每次实例化对象的时候，都会重新调用一次`list`方法生成一个新的空列表给`nums`,从而符合我们的预期。

## 3. 使用`dataclasses.asdict`转换对象到`dict`

我们经常会遇到需要持久化复杂数据对象的情况，比如存到数据库或者转化为`json`输出到前端。如果能方便的将对象转换成`dict`的话会很大的提高开发效率，幸运的是标准库提供了`dataclasses.asdict`函数。

我们来定义一个稍微复杂一点的数据类

```python
@dataclass
class A:
    a:int

@dataclass
class B:
    name:str
    a_list:List[A]
```

如果要手动把`B`类型对象转化为`dict`的话，我们大概要这样做：

```python
def to_dict(self):
    return {'name':self.name, 'a_list':[{'a':x.a} for x in self.a_list]}
```

这样的`to_dict`方法偶尔实现一次两次倒也罢了，如果每个数据类都要手动实现一个`to_dict`方法的话就太过浪费时间和精力了。使用`dataclasses.asdict`方法可以极大地提高我们的效率。

```python
In [1]: from dataclasses import dataclass,asdict

In [2]: from typing import List

In [3]: @dataclass
   ...: class A:
   ...:     a:int
   ...:
   ...: @dataclass
   ...: class B:
   ...:     name:str
   ...:     a_list:List[A]
   ...:

In [4]: b = B(name='b',a_list=[A(1),A(2),A(3)])

In [5]: b
Out[5]: B(name='b', a_list=[A(a=1), A(a=2), A(a=3)])

In [6]: asdict(b)
Out[6]: {'name': 'b', 'a_list': [{'a': 1}, {'a': 2}, {'a': 3}]}
```

## 4. 使用`__post_init__`方法进行一些初始化操作

使用`dataclass`装饰的类一个主要优势就是不用手动去实现`__init__`方法，但我们经常需要在对象初始化的时候对一些数据进行校验或者额外操作，此时一个选择是手动实现`__init__`方法，其中会有大段的模板代码例如`self.a=a;self.b=b`，另一个选择是定义`__post_init__`方法来进行初始化操作，例如：

```python

In [1]: from dataclasses import dataclass

In [2]: @dataclass
   ...: class Person:
   ...:     name:str
   ...:     age:int
   ...:
   ...:     def __post_init__(self):
   ...:         if self.age < 0:
   ...:             raise ValueError('Age < 0')
   ...:

In [3]: Person('Mike',-1)
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-3-4b4379de6be2> in <module>
----> 1 Person('Mike',-1)

<string> in __init__(self, name, age)

<ipython-input-2-1e8016803a1d> in __post_init__(self)
      6     def __post_init__(self):
      7         if self.age < 0:
----> 8             raise ValueError('Age < 0')
      9

ValueError: Age < 0
```

## 总结

今天向大家介绍了 Python 3.7 中`dataclasses`标准库的简单使用。
作为一个以灵活著称的编程语言，我们使用 Python 处理结构化数据的时候经常会使用`dict`在不同模块间传来传去，然后在需要的地方进行数据的校验和格式转换，无意间会增加很多相似的代码，真正的核心逻辑淹没在这些校验和转换过程中，单元测试的复杂度也随之提高。
利用新版本 Python 的类型提示和 dataclasses 标准库，配合一些开发工具（mypy,pylint 等）我们可以较为放心地将数据对象在不同模块方法间传递使用，有助于改进 Python 在大型项目下的开发效率和安全性。
