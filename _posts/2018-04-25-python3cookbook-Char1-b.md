---
layout:     post
title:      "Python3 Cookbook char1-b"
subtitle:   "python3 cookbook 学习笔记"
date:       2018-04-25 12:00:00
author:     "JM"
header-img: "img/post-bg-galaxy.jpg"
tags:
    - Python3 Cookbook
    - 2018-04
---

# Char1 数据结构与算法 b

## 1.11 切片命名

切片对象：通过内置slice()创建，可以减少切片下标使用


```python
record = list(range(1,21))
odd = slice(0,20,2)
even = slice(1,20,2)
print("Odd num:" + str(record[odd]))
print("Even num" + str(record[even]))
```

    Odd num:[1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
    Even num[2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
    

切片对象有slice.start, slice.stop, slice.step等属性，这意味着切片对象只能是**等差数列**的形式；
切片对象不会出现IndexError即使超出index范围：


```python
trip = slice(2,30,3)
print(trip.start, trip.stop, trip.step)
print("Triple num:" + str(record[trip]))
```

    2 30 3
    Triple num:[3, 6, 9, 12, 15, 18]
    

## 1.12 统计元素出现的次数

由hashable对象组成的序列对象，可以使用collections.Counter 类：

频率最高的几个元素可通过most_common()方法：


```python
words = [
    'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
    'eyes', "don't", 'look', 'around','my', 'eyes'
]

from collections import Counter
w_counts = Counter(words)
top_3 = w_counts.most_common(3)

print(w_counts)
print(top_3)
print(w_counts['my'])
```

    Counter({'eyes': 4, 'look': 3, 'my': 3, 'into': 2, "don't": 1, 'around': 1})
    [('eyes', 4), ('look', 3), ('my', 3)]
    3
    

从上面看出，most_common()返回的是tuple组成的list；而Counter对象底层是dict.

**更新Counter对象**可以像对待dict一样使用手动或者使用update()


```python
morewords = ['look', 'into', 'your', 'eyes',]
# use update()
w_counts.update(morewords)
print(w_counts)

# use for loop
for i in morewords:
    w_counts[i] += 1
```

    Counter({'eyes': 5, 'look': 4, 'into': 3, 'my': 3, "don't": 1, 'around': 1, 'your': 1})
    

Couter对象还支持直接加减操作，加操作相当于update, 减法操作相当于去掉某些元素：


```python
a = Counter(words)
b = Counter(morewords)
c = a - b
print(c)
```

    Counter({'my': 3, 'eyes': 3, 'look': 2, 'into': 1, "don't": 1, 'around': 1})
    

## 1.13 字典组成的列表排序

### sorted()函数：

sorted(iterable, *, key=None, reverse=False) 

返回一个排序后的iterable中元素组成的list.

list.sort()和 sorted()都有key parameter(形参)， key应该指定为一个函数（callable对象），用来在每个元素上调用：

例如：对一个元组中各个元素不分大小写按字母排序：


```python
 sorted( ("This", "is", "a", "Test"), key=str.lower)
```




    ['a', 'is', 'Test', 'This']



key函数只能接受一个argument，返回值用于排序，常用**lambda**匿名函数：


```python
student_tup = [
    ('john','A',15),
    ('kat','B',30),
    ('bob','B',12),
    ('ella','B',11)
]
sorted(student_tup, key=lambda student: student[2])
```




    [('ella', 'B', 11), ('bob', 'B', 12), ('john', 'A', 15), ('kat', 'B', 30)]



### operator 模块下的 itemgetter()


```python
from operator import itemgetter
print(sorted(student_tup,key=itemgetter(2)))
```

    [('ella', 'B', 11), ('bob', 'B', 12), ('john', 'A', 15), ('kat', 'B', 30)]
    

定义 f = itemgetter(2), 再对实例R调用 f(R),返回 R[2];

定义 g = itemgetter(2,3,4), 再对实例R调用 g(R),返回元组 (R[2],R[3],R[4])

相当于：

```python
def item_g(*items):
  if len(items) = 1:
    item = items[obj]
    def g(obj):
      return obj[item]

  else:
    def g(obj):
      return tuple(obj[item] for item in items)
  
  return g
```     

源代码：
```python
class itemgetter:
    """
    Return a callable object that fetches the given item(s) from its operand.
    After f = itemgetter(2), the call f(r) returns r[2].
    After g = itemgetter(2, 5, 3), the call g(r) returns (r[2], r[5], r[3])
    """
    __slots__ = ('_items', '_call')

    def __init__(self, item, *items):
        if not items:
            self._items = (item,)
            def func(obj):
                return obj[item]
            self._call = func
        else:
            self._items = items = (item,) + items
            def func(obj):
                return tuple(obj[i] for i in items)
            self._call = func

    def __call__(self, obj):
        return self._call(obj)

    def __repr__(self):
        return '%s.%s(%s)' % (self.__class__.__module__,
                              self.__class__.__name__,
                              ', '.join(map(repr, self._items)))

    def __reduce__(self):
        return self.__class__, self._items
```

对于有\__getitem\__方法的对象，都可调用itemgetter()；如student_tup列表的第一个元素：


```python
itemgetter(0,2)(student_tup[0])
```




    ('john', 15)



回到小节标题，有个字典组成的列表,按照两列排序：


```python
rows = [
    {'fname': 'Brian', 'lname': 'Jones', 'uid': 1005},
    {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]

sorted(rows, key = itemgetter("lname","uid"))
```




    [{'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
     {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
     {'fname': 'Big', 'lname': 'Jones', 'uid': 1004},
     {'fname': 'Brian', 'lname': 'Jones', 'uid': 1005}]



## 1.14 比较不支持原生比较的对象

如果按照实例的属性进行排序：

例如：列表中的每个元素都是Student实例，按照grade,name排序,利用lambda：


```python
class Student:
    def __init__(self, name, grade, age):
        self.name = name
        self.grade = grade
        self.age = age
    def __repr__(self):
        return repr("{}-{}:{}".format(self.name, self.age, self.grade))
    
student_objects = [
     Student('john', 'A', 15),
     Student('kat', 'B', 30),
     Student('bob', 'B', 12),
     Student('ella','B', 11)
]

sorted(student_objects, key=lambda student: (student.grade, student.age))
```




    ['john-15:A', 'ella-11:B', 'bob-12:B', 'kat-30:B']



### operator模块下attrgetter()


```python
from operator import attrgetter
print(sorted(student_objects, key = attrgetter('age')))
```

    ['ella-11:B', 'bob-12:B', 'john-15:A', 'kat-30:B']
    

## 1.15 将记录分组

### itertools 模块的 groupby()

itertools.groupby(iterable, key=None)

不指定key时候，默认iterable中的元素本身参与分组：


```python
from itertools import groupby

for key, items in groupby('aabbbcccc'):
    print(key, list(items))
```

    a ['a', 'a']
    b ['b', 'b', 'b']
    c ['c', 'c', 'c', 'c']
    

key参数指定iterable中元素的参与分组时要做的变换：

指定key为lambda匿名函数：


```python
for key, items in groupby('aaABBbCcCc', key=lambda x: x.lower()):
    print(key, list(items))
```

    a ['a', 'a', 'A']
    b ['B', 'B', 'b']
    c ['C', 'c', 'C', 'c']
    

**使用groupby()之前应该确保排过序**

设置key参数为itemgetter()


```python
rows = [
    {'address': '5412 N CLARK', 'date': '07/01/2012'},
    {'address': '5148 N CLARK', 'date': '07/04/2012'},
    {'address': '5800 E 58TH', 'date': '07/02/2012'},
    {'address': '2122 N CLARK', 'date': '07/03/2012'},
    {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
    {'address': '1060 W ADDISON', 'date': '07/02/2012'},
    {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
    {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
]

rows_by_date = sorted(rows, key = itemgetter('date','address'))

from itertools import groupby

for key, items in groupby(rows_by_date, key=itemgetter('date')):
    print(key)
    for x in items:
        print("--",x)
```

    07/01/2012
    -- {'address': '4801 N BROADWAY', 'date': '07/01/2012'}
    -- {'address': '5412 N CLARK', 'date': '07/01/2012'}
    07/02/2012
    -- {'address': '1060 W ADDISON', 'date': '07/02/2012'}
    -- {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'}
    -- {'address': '5800 E 58TH', 'date': '07/02/2012'}
    07/03/2012
    -- {'address': '2122 N CLARK', 'date': '07/03/2012'}
    07/04/2012
    -- {'address': '1039 W GRANVILLE', 'date': '07/04/2012'}
    -- {'address': '5148 N CLARK', 'date': '07/04/2012'}
    


### 与collection.defaultdict()

如果只是想根据记录的某些字段分配到更大的数据结构，并且允许随机访问, 使用defaultdict将分组用的key作为dict的键，同一组内的各个元素放到列表


```python
from collections import defaultdict

rows_multidict = defaultdict(list)
for row in rows:
    rows_multidict[row['date']].append(row)

# retrive a group
rows_multidict['07/01/2012']
```




    [{'address': '5412 N CLARK', 'date': '07/01/2012'},
     {'address': '4801 N BROADWAY', 'date': '07/01/2012'}]



参考资料：http://python3-cookbook.readthedocs.io/zh_CN/latest/chapters/p01_data_structures_algorithms.html
