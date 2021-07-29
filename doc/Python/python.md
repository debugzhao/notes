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

