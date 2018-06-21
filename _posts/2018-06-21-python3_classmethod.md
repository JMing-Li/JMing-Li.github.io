---
layout:     post
title:      "Python3 Class Methods & Static Methods"
subtitle:   "理解 类方法 和 静态方法"
date:       2018-06-21 12:00:00
author:     "JM"
header-img: "img/post-bg-leopard.jpg"
tags:
    - Python3
---
## Class Methods & Static Methods

类方法和静态方法都是基于描述器从而实现的，如何实现将在理解descriptor后理解，先看看这两种方法的使用：  

@classmethod means: when this method is called, we pass the class as the first argument instead of the instance of that class (as we normally do with methods). This means you can use the class and its properties inside that method rather than a particular instance.    
@staticmethod means: when this method is called, we don't pass an instance of the class to it (as we normally do with methods). This means you can put a function inside a class but you can't access the instance of that class (this is useful when your method does not use the instance).

### Class Methods

当我们只想和类交互，而不依赖实例的时候可以使用类方法：  
下面的例子，记录Student类调用的次数，如果调用这个类一次保存为一个student实例，则记录了实例的个数。  
当不使用class method:


```python
def get_student_num(cls):
    return cls.num
# get_student_num函数接受类名称作为参数，返回该类的num属性

class Student:
    num = 0
    def __init__(self, name):
        Student.num += 1
        self.school = 'SJTU'
        self.name = name
```


```python
s1 = Student('Bob')
s2 = Student('Jack')
get_student_num(Student)
```




    2



get_student_num函数用来获取Student类的属性，却定义在类定义代码之外，不利于维护。  
下面使用class method将该函数定义到类代码中：


```python
class Student:
    num = 0
    def __init__(self, name):
        Student.num += 1
        self.school = 'SJTU'
        self.name = name
        
    @classmethod
    def get_student_num(cls):
        return cls.num
```


```python
s1 = Student('Mike')
s2 = Student('Joy')
s3 = Student('Jone')
Student.get_student_num()
```




    3



classmethod不仅可以通过类，还可以通过实例来调用：


```python
s3.get_student_num()
```




    3



### Static Methods

有一些跟类的功能相关，但是不与要类及实例参与的情况，比如更改环境变量，这需要使用静态方法。  



```python
IND = 'ON'
class Kls(object):
    def __init__(self, data):
        self.data = data
        
    @staticmethod
    def checkind():    #注意这个函数没有参数
        return (IND == 'ON')
    
    def set_db(self):
        if self.checkind():
            self.db = 'New db connection'
        print('DB connection made for: ', self.data, self.db)
               
ik1 = Kls(121)
```

静态方法没有参数，调用时不需要类与实例的参与，没有self或cls参数传入。


```python
ik1.checkind()
```




    True



我的思考：上面的静态方法可以用一般方法替代，传入self参数，但是函数中并不使用self


```python
IND = 'ON'
class Kls(object):
    def __init__(self, data):
        self.data = data
        
    def checkind(self):    ## 一般的实例方法
        return (IND == 'ON')
    
    def set_db(self):
        if self.checkind():
            self.db = 'New db connection'
        print('DB connection made for: ', self.data, self.db)
        
k1 = Kls(100)
```


```python
k1.checkind()
```




    True




```python
k1.set_db()
```

    DB connection made for:  100 New db connection
    

### 比较参数

分别使用实例，类，来调用静态方法，类方法


```python
class Kls:
    def __init__(self, data):
        self.data = data
    def printd(self):
        print(self.data)
    @staticmethod
    def smethod(*arg):
        print('Static:', arg)
    @classmethod
    def cmethod(*arg):
        print('Class:', arg)

k1 = Kls(111)
```


```python
k1.printd()
k1.smethod()
k1.cmethod()
```

    111
    Static: ()
    Class: (<class '__main__.Kls'>,)
    

由此可见，实例调用classmethod传入的参数是：实例所属的类。


```python
# Kls.printd()  
## TypeError: printd() missing 1 required positional argument: 'self'
```


```python
Kls.smethod()
Kls.cmethod()
```

    Static: ()
    Class: (<class '__main__.Kls'>,)
    

### 更多例子

转载自stackoverflow:  

假设我们想要建立很多MyData class的实例，现在有日期形式的字符串'dd-mm-yy',我们需要：  
 1. 将字符串解析成三个整数变量或者包含这三个元素的元组  
 2. Instantiate MyData by passing those values to initialization call.  
 
利用classmethod可以完美实现这些要求：  
下面的classmethod 'from_string'相当于一个构造器，具有以下优势:    
 1. 实现解析字符串，并且MyData.from_string这个函数可以复用  
 2. 闭包很好（如果想将解析字符这个函数单独用到别处，也符合OOP规范  
 3. cls对象保持着MyData这个类本身，而非类的一个实例，如果继承MyData类，子类也会有这个函数。


```python
class MyDate:

    def __init__(self, day=0, month=0, year=0):
        self.day = day
        self.month = month
        self.year = year

    @classmethod
    def from_string(cls, date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        date1 = cls(day, month, year)
        return date1

    @staticmethod
    def is_date_valid(date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        return day <= 31 and month <= 12 and year <= 2019
```

再看staticmethod, 'is_data_valid',这个方法不用在乎MyData类或是其实例，就像一个普通的函数，只是一个语法上的方法，而不进入MyData类或实例的内部.


```python
d1 = MyDate.from_string('20-11-2017')
```


```python
d1.__dict__
```




    {'day': 20, 'month': 11, 'year': 2017}




```python
MyDate.is_date_valid('33-12-2016')
```




    False



references:  
https://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner/12179325#12179325  
https://www.zhihu.com/question/20021164/answer/42704772  

