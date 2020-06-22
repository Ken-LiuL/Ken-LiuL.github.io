---
title: Python CookBook
date: 2020-05-07 15:31:33
tags: Python
categories: 读书笔记
---
Python是一门非常好作为入门学习的编程语言，语言简洁干练，拥有强大的第三方库，可以用来做机器学习、数据分析、爬虫等等。但是，很多人入门之后对Python的认知就停留在了比较浅显的认知上，成为了调包侠， Python本身有一些特性和库（如collections、future、generator等）是值得并且需要仔细来琢磨的。[Python Cookbook](https://book.douban.com/subject/4828875/) 就是一本可以作为进阶学习Python的书籍，里面的很多技巧和方法论非常实用。
<!-- more -->

# 数据结构和算法
## 迭代器
迭代器有两个基本的方法：iter()和next()。我们可以实现自己的迭代器，通过实现__iter__和__next__方法。
```python
class MyNumbers:
    def __iter__(self):
        self.start = 0
        return self
    def __next__(self):
        self.start += 1
        if self.start > 10:
            raise StopIteration
my_number = MyNumbers()
my_iter = iter(my_number)
for x in my_iter:
    print(x)
```    
## 序列分解
任何的序列（或者是可迭代对象）可以通过一个简单的赋值操作来分解为单独的变量, 如果变量总数和序列里的元素数目不匹配会抛出ValueError，不过我们可以用*操作符来处理这种情况。
```Python
>>> a = [1, 2 , 3]
>>> x, y, z = a
>>> x
1
>>> x, y = a
ValueError: too many values to unpack (expected 2)
>>> *x, y = a
>>> x
[1, 2]
>>> x, *_, y = a  #如果你想丢弃部分元素
>>> x
1
>>> y
3
```
## Deque
Deque可以以O(1)的时间复杂度在头部和尾部进行append和pop的操作，既可以作为queue也可以作为stack来使用。
> Deques support thread-safe, memory efficient appends and pops from either side of the deque with approximately the same O(1) performance in either direction

```Python
>>> from collections import deque
>>> q = deque(maxlen=3)
>>> q.append(1)
>>> q.append(2)
>>> q.appendleft(3)
>>> q
deque([3,1,2])
>>> q.pop()
2
>>> q.popleft()
3
```
## Heapq
Heapq就是堆数据结构（也叫优先队列），他满足**heap[k] <= heap[2*k+1] and heap[k] <= heap[2*k+2]**, heapq是最小堆， heapq[0]得到的总是最小值。
```python
import heapq
nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums)) # Prints [42, 37, 23]
print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]
#优先队列
class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0
    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, item, self._index))
        self._index += 1
    def pop(self):
        return heapq.heappop(self._queue)[-1]
```
heap[0]和heapq.heappop()操作永远得到当前最小的元素，每次操作的时间复杂度仅仅为logN，所以当需要得到n个最小的元素并且n远远小于列表的长度的时候，通过heapq来实现是比较好的。

## 字典
常常我们想要一个key对应多个值，容易写出
```python
d = {}
if v in d:
    d[v].append()
else:
    d[v] = []
#或者
d.setdefault(v, []).append()
```
这样的代码块，setdefault相对方便一点，但是每次都要调用很繁琐，其实我们可以直接用**defaultdict**
```python
from collections import defaultdict
d = defaultdict()
d['test'].append('test')
```
而有的时候当我们迭代字典的时候希望能够保留插入的顺序，那么可以使用**OrderedDict**
```python
from collections import OrderedDict
d = OrderedDict()
d['test1'] = 1
d['test2'] = 2
for k, v in d.items():
    print(k, v)
```
如果要比较字典的键值的相同或者不同，也有方便的操作符
```python
a.keys() & b.keys()   #intersection
a.keys() - b.keys()   #difference
a.items() & b.items()
a.items() - b.items()
```

