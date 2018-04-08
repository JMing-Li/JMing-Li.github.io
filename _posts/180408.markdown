---
layout:     post
title:      "Hello python3"
subtitle:   "Python3 Cookbook notes"
date:       2018-04-08
author:     "JM"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - python
---

 

## Char 1 数据结构和算法

### 1.1-1.2 解压可迭代对象赋值给多个变量 
使用星号*，从可迭代对象中提取一些元素来赋值:

```python
mlist = list(range(10))
print(mlist)
```

    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    
例如丢弃开头的2个和最后的2个，只提取中间的一些：

```python
ign,ign, *ele, ign, ign = range(10)
print(ele)
print(ign)
```

    [2, 3, 4, 5, 6, 7]
    9
    

### 1.4-1.5 堆数据
heap.heapify（）在原位将list转为heap,
heap[0]总是最小的元素（其他元素无序！）：

```python
import heapq
nums = [2,4,1,-4,99,-99,30,-40]
```


```python
hp = list(nums)
heapq.heapify(hp)
print(hp)      
```

    [-99, -40, 1, -4, 99, 2, 30, 4]
    
heapq.pop() pop出最小的那个，heapq.heappush() push进堆；
且原来的堆数据就被修改了：

```python
heapq.heappush(hp, -1000)
heapq.heappop(hp)
```




    -1000


最大或最小的几个元素：

```python
heapq.nlargest(2,hp)
```




    [99, 30]



### 1.6 字典
字典中一个key对应多个value

```python
from collections import defaultdict
d = defaultdict(list)
d['a'].append(1)
d['a'].append(2)
d['b'].append(4)
```


```python
d
```




    defaultdict(list, {'a': [1, 2], 'b': [4]})




```python
d = defaultdict(set)
d['a'].add(1)
d['a'].add(2)
d['b'].add(4)
d
```




    defaultdict(set, {'a': {1, 2}, 'b': {4}})



### 1.10 去除重复且保持原顺序 
通过set来消除重复会改变原来的顺序，通过yield构造生成器可保留原顺序：

```python
def dedupe(items):
    seen = set()
    for item in items:
        if item not in seen:
            yield item
            seen.add(item)
            
a = [1,2,3,1,2,5]
list(dedupe(a))
```




    [1, 2, 3, 5]


构造生成器返回一个列表中所有3的倍数：

```python
def get3(lst):
    for item in lst:
        if item % 3 == 0:
            yield item
            
get3(range(20))
```




    <generator object get3 at 0x000002272D0B96D0>




```python
list(get3(range(10)))
```




    [0, 3, 6, 9]


对于dict等非harshbale类型，需要额外函数来处理：

```python
a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
```


```python
def dedupe(items, func = None):
    seen = set()
    for i in items:
        val = i if func is None else func(i)
        if val not in seen:
            yield i
            seen.add(val)
            
list(dedupe(a, func = lambda d: (d['x'],d['y'])))
```




    [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]



