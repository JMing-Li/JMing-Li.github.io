---
layout:     post
title:      "Python3 迭代器 生成器"
subtitle:   "python3 学习笔记"
date:       2018-04-26 12:00:00
author:     "JM"
header-img: "img/post-bg-simple2.jpg"
tags:
    - Python3 
    - 2018-04
---

## 理解迭代器生成器

### 3-2 迭代器

iterable :  
  - 可以迭代的对象（比如字符，文件），
  - 有__iter__method 能返回 iterator， 或者有__getitem__ method.相当于容器。

iterator :  
  - 在迭代中记住状态，
    - 有__next__method
    - 能返回下一个值，
    - 能更新状态指向下一值，
    - 当完成所有迭代时 raise StopIteration.
  - 自身 self-iterable (自身有__iter__ method返回self)。
    
Note：  
- \__next\__ method 在python2 为 next method
- The builtin function next() calls that method on the object passed to it.



```python
import requests

from collections import Iterable, Iterator

class WeatherIterator(Iterator):
    def __init__(self, cities):
        self.cities = cities
        self.index = 0
        
    def getweather(self, city):  
        r = requests.get(u'http://wthrcdn.etouch.cn/weather_mini?city=' + city) #unicode
        data = r.json()['data']['forecast'][0]
        return 'low-{1} high-{2},{0}'.format(city, data['low'],data['high'])
    
    def __next__(self):
        if self.index == len(self.cities):
            raise StopIteration
        city = self.cities[self.index]
        self.index += 1
        return self.getweather(city)
    
class WeatherIterable(Iterable):
    def __init__(self, cities):
        self.cities = cities
        
    def __iter__(self):
        return WeatherIterator(self.cities)
    
for x in WeatherIterable([u'南京',u'苏州',u'扬州']): print(x)  
```

    low-低温 14℃ high-高温 23℃,南京
    low-低温 13℃ high-高温 22℃,苏州
    low-低温 14℃ high-高温 22℃,扬州
    

### 3-3 生成器   
yeild :使用生成器函数实现可迭代对象  
Q：  
实现一个可迭代对象的类，它能够迭代出给定范围内的所有质数。  
A：  
将该类的__iter__ method实现成生成器，每次yield返回一个质数。


```python
class PrimeNumbers:
    def  __init__(self, start, end):
        self.start = start
        self.end = end
        
    def isPrime(self, k):
        if k < 2: return False
        
        for i in range(2,k):
            if k % i == 0: return False
            
        return True
    
    def __iter__(self):
        for k in range(self.start, self.end +1):
            if self.isPrime(k):
                yield k
                
a = PrimeNumbers(10,40)

for x in a: print(x)
```

    11
    13
    17
    19
    23
    29
    31
    37
    
