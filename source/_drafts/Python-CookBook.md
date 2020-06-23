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
如果要比较字典的键值的相同或者不同，也有方便的集合操作符
```python
a.keys() & b.keys()   #intersection
a.keys() - b.keys()   #difference
a.items() & b.items()
a.items() - b.items()
```
需要注意的是**a.values()**方法并不支持这样的集合操作。

## 命名切片
在做数据处理的时候，常常需要频繁的对序列进行切片，满屏幕的切片操作常常会让代码可读性大大降低，命名切片可以提高代码可读性和维护性。
```python
>>> a = slice(0, 10, 2) #start, stop, step
>>> b = [1,2,3,4,5]
>>> b[a]
[1,3,5]
>>> a.indices(5)       #适配一个合适的范围
(0, 5, 2)
```
## 制表或者计数
对于字典而言，一个常见的用法是，我们经常用来统计元素的频率
```python
for e in dic:
    if e in dic:
        dic[e] += 1
    else:
        dic[e] = 1
```
不过其实python的**Counter**可以直接用来解决此类问题
```python
from collections import Counter
a = [1,2,3,4,3]
fre = Counter(a)
fre.most_common(3)          #top 3 出现的元素
#一个有意思的地方是，Counter可以直接做数学操作
fre2 = Counter([1,2,3,4])
fre - fre2                 #difference
fre + fre2                 #union
```
## 字典和对象排序
如果需要对数组字典进行排序的话，我们常常的做法是
```python
sorted(data, key=lambda...)
```
python其实提供一个专门的函数来处理这种情况，**itemgetter**
```python
from operator import itemgetter
sroted(data, key=itemgetter('field1', 'field2'))
#min,max也可以使用
max(l, key=itemgetter('field1))
```
类似得，如果是对数组对象进行排序的话，我们可以使用**attrgetter**
```python
from operator import attrgetter
sorted(users, key=attrgetter('user_id'))
```
值得注意是，虽然两种做法都可以，但是来自operator的做法会更快一点
## 数据分组
常用的数据分组我们会用pandas来进行，其实python的itertools包也提供类似的功能
```python
from operator import itemgetter
from itertools import groupby
data.sort(key=itemgetter('user_id'))
groupby(data, key=itemgetter('user_id'))
```
值得注意的地方是，**groupby**的逻辑是检查连续的元素，所以事先我们需要对数据进行第一轮排序。
## 数据过滤
数据过滤有很多做法，除去简单的for loop，我们可以使用过滤器、迭代器、生成器以及itertool里面的工具
```python
[n for n in l if n > 0]        #list comprehension
(n for n in l if n > 0)        #generator 
list(filter(lambda x:.., l))   #filter
[n if n >0 else -1 for n in l] #transformation

from itertools import compress
a = [1,2,3,4,5]
b = [ i > 5 for i in a]
list(compress(a, b))            #根据b中的布尔值过滤对应的元素 
```
## namedtuple
当需要频繁大量的通过下标来访问tuple的时候，我们可以使用namedtuple来提高程序的可读性
```python
>>> from collections import namedtuple
>>> User = namedtuple('User', ['user_id', 'username'])
>>> u = User('no1', 'jacky')
>>> u.username
jacky
```
## 合并字典
合并字典也是常见的操作之一，我们经常会写if代码块来做这样的工作，或者使用update方法，这样会创建一个新的字典，但是其实可以直接使用 ChainMap来处理字典的合并查询，合成的新对象只是两个字典逻辑上的合并，并没有新的字典对象被创造出来
```python
a = {'x':1, 'z':3}
b = {'y':2, 'z':4}
b.update(a)        #用a来覆盖b，相同的key的话，会采用a中的值  

from collections import ChainMap
a = {'x':1, 'z':3}
b = {'y':2, 'z':4}
c = ChainMap(a, b)   #用a来覆盖b

#ChainMap一个有意思的地方是可以很方便的作为编程语言作用域的处理
env = ChainMap()
env['x'] = 1
env = env.new_child()
env['x'] = 2
env = env.new_child()
#现在env['x']等于2
env = env.parents
#现在env['x']等于1
```

