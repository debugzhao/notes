#### 函数

##### 空函数

实际上`pass`可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来

```python
def nop():
    pass
```

##### 返回多个值

```python
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
```

##### 默认参数

*定义默认参数要牢记一点：默认参数必须指向不变对象！*

```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```

##### 可变参数

```python
# 在函数内部，参数numbers接收到的是一个tuple
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```

#### 高级特性

##### 切片

```python
L[开始下标 : 结束下标 : 步长]
# 左闭右开
```

##### 迭代

如何判断一个对象是可迭代对象呢？方法是通过`collections.abc`模块的`Iterable`类型判断：

```python
>>> from collections.abc import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

1. 字典

   ```python
   >>> d = {'a': 1, 'b': 2, 'c': 3}
   >>> for key in d:
   ...     print(key)
   ...
   a
   c
   b
   ```

2. 字符串

   ```python
   >>> for ch in 'ABC':
   ...     print(ch)
   ...
   A
   B
   C
   ```

3. 获取迭代元素的下标

   ```python
   >>> for i, value in enumerate(['A', 'B', 'C']):
   ...     print(i, value)
   ...
   0 A
   1 B
   2 C
   ```

4. 同时迭代两个元素

   ```python
   >>> for x, y in [(1, 1), (2, 4), (3, 9)]:
   ...     print(x, y)
   ...
   1 1
   2 4
   3 9
   ```

##### 列表生成式

1. 普通列表生成式

   ```python
   >>> [x * x for x in range(1, 11)]
   [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
   ```

2. 带判断的列表生成式

   ```python
   >>> [x * x for x in range(1, 11) if x % 2 == 0]
   [4, 16, 36, 64, 100]
   ```

3. 双重循环列表生成式

   ```python
   >>> [m + n for m in 'ABC' for n in 'XYZ']
   ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
   ```

##### 生成器

如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。

要创建一个generator，有很多种方法。第一种方法很简单，只要把一个列表生成式的`[]`改成`()`，就创建了一个generator：

```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>

>>> next(g)
0
>>> next(g)
1
>>> next(g)
4
```

**如果一个函数定义中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator：**

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

#### 模块

##### 使用模块

1. 使用模块

   ```python
   #!/usr/bin/env python3
   # -*- coding: utf-8 -*-
   
   ' a test module '
   
   __author__ = 'Michael Liao'
   
   import sys
   
   def test():
       args = sys.argv
       if len(args)==1:
           print('Hello, world!')
       elif len(args)==2:
           print('Hello, %s!' % args[1])
       else:
           print('Too many arguments!')
   
   if __name__=='__main__':
       test()
   ```

2. 作用域

   外部不需要引用的函数全部定义成private，只有外部需要引用的函数才定义为public。

   类似`_xxx`和`__xxx`这样的函数或变量就是非公开的（private），不应该被直接引用，比如`_abc`，`__abc`等；

##### 安装第三方模块



#### 小技巧

##### 检查参数

```python
def my_abs(x):
    if not isinstance(x, (int, float)):
        raise TypeError('bad operand type')
    if x >= 0:
        return x
    else:
        return -x
```

