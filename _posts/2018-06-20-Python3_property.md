---
layout:     post
title:      "python3 property"
subtitle:   "理解 property()"
date:       2018-06-20 12:00:00
author:     "JM"
header-img: "img/post-bg-leopard.jpg"
tags:
    - Python3
---
# 理解 python3 property()

class property(fget=None, fset=None, fdel=None, doc=None)  
property可以给实例的attribute增加除了访问，修改之外的其他处理逻辑，比如增加合法性检查等.  

## 一、property的一个pure python实现

property的一个pure python equivalent如下：  
包含**描述器部分**和**装饰器部分。**


```python
class PProperty(object):
    "Hi! Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc
    
    # descriptor part :
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("op! unreadable attribute")
        print('Hi! It is descriptor get')
        return self.fget(obj)
        
    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("op! can't set attribute")
        print('Hi! It is descriptor set')
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("op! can't delete attribute")
        print('Hi! It is descriptor delete')
        self.fdel(obj)

    # decorator part :
    def getter(self, fget):
        print('yo! decorator getter')
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        print('yo! decorator setter')
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        print('yo! decorator deleter')
        return type(self)(self.fget, self.fset, fdel, self.__doc__)
```

### 1，通过函数调用property

通过 **property(fget=None, fset=None, fdel=None, doc=None)** 函数的方式调用property:  
调用方法：  
\_\_init\_\_ 设置内部attribute；  
定义三个方法来操作内部attribute；  
property()函数设置外部可访问的变量property.    


```python
class C(object):
    def __init__(self): 
        self.__x = 100
    def getx(self):
        print('here is class c getx')
        return self.__x
    def setx(self, value): 
        print('here is class c setx')
        self.__x = value
    def delx(self): 
        print('here is class c delx')
        del self.__x
    x = PProperty(getx, setx, delx, "Woo!!! I'm the 'x' property.")
    y = 'contr!!!'
    
obj1 = C()
```

**class C定义的时候，将property实际操纵的内部属性，操纵的3个方法，外部“马甲”变量，都打包在这里**

这样class C就包含了一个可控制的外部变量: property x.  
property x 其实是内部attribute \_\_x的“马甲”.  
property只是访问，修改，删除内部attribute的外部接口。实例\_\_dict\_\_中只有内部attribute:  


```python
obj1.__dict__
```




    {'_C__x': 100}




```python
obj1.x = 101
```

    Hi! It is descriptor set
    here is class c setx
    

![PProperty](/img/in-post/property.png)

**赋值操作实现流程**：  
上图描述出类PProperty,类C，以及实例obj1赋值后的关系。当实例obj1初始化时，obj1.\_C\_\_x 值为100；  

obj1.x = 1212    ## x为判断为 data descriptor   
--》 x.\_\_set\_\_(self, obj, value)  ==》  x.\_\_set\_\_(x,obj1,1212)  ## \_\_set\_\_的self是PProperty实例。   
--》 print('Hi! It is descriptor set')  
--》 self.fset(obj, value) ==》 x.fset(obj1,1212) ==》 setx(obj1,1212)  ## setx的self是 class C实例  
--》 print('here is class c setx')  
--》 self.\_\_x = value  ==》  obj1.\_\_x = 1212  ==》 Return value: None  
--》 self.fset(obj. value)执行完毕，修改值完成.  

print(obj1.x)  
--》 x.\_\_get__(self, obj, objtype) ==》 x.\_\_get\_\_(x, obj1, None)  
--》 print('Hi! It is descriprot get')  
--》 self.fget(obj)  ==》  x.fget(obj1)  ==》 getx(obj1)   
--》 self.\_\_x  ==》  obj1.\_\_x 执行完毕.  


通过property()函数调用只用到了PProperty类的\_\_init\_\_部分和描述器部分，而用不到装饰器部分：  


```python
print(obj1.x)
```

    Hi! It is descriptor get
    here is class c getx
    101
    

通过property x 实现对内部\_\_x的访问，修改，删除，检查等：  


```python
del obj1.x
```

    Hi! It is descriptor delete
    here is class c delx
    

而实例obj1中的另外一个普通属性y只可以通过obj1.y访问，无其他操作。  
实例obj1还获得了property x 的\_\_doc\_\_ 信息：


```python
help(obj1)
```

    Help on C in module __main__ object:
    
    class C(builtins.object)
     |  Methods defined here:
     |  
     |  __init__(self)
     |      Initialize self.  See help(type(self)) for accurate signature.
     |  
     |  delx(self)
     |  
     |  getx(self)
     |  
     |  setx(self, value)
     |  
     |  ----------------------------------------------------------------------
     |  Data descriptors defined here:
     |  
     |  __dict__
     |      dictionary for instance variables (if defined)
     |  
     |  __weakref__
     |      list of weak references to the object (if defined)
     |  
     |  x
     |      Woo!!! I'm the 'x' property.
     |  
     |  ----------------------------------------------------------------------
     |  Data and other attributes defined here:
     |  
     |  y = 'contr!!!'
    
    

### 2，通过Syntactic Sugar调用property

使用语法糖@实际使用的是PProperty类的装饰器部分先进行包装:  
使用下面的语法，指定了PProperty()函数的fget, fset, fdel三个参数。


```python
class CC:
    def __init__(self):
        self._x = 200
    
    # getter function
    @PProperty    # PProperty.getter()包装，并未调用
    def x(self):
        """lala ! I'm the 'x' property."""
        print('hi class CC x PProperty')
        return self._x
    
    # setter function
    @x.setter   # 调用了PProperty.setter()
    def x(self, value):
        self._x = value
        print('hi class CC x setter')

    # deleter function
    @x.deleter
    def x(self):
        del self._x
        print('hi class CC x deleter')
```

    yo! decorator setter
    yo! decorator deleter
    

定义class CC的时候就调用了装饰器部分进行包装。  
装饰器getter方法没有调用，setter和deleter方法被调用 ？


```python
obj2 = CC()
```


```python
print(obj2.x)
```

    Hi! It is descriptor get
    hi class CC x PProperty
    200
    

## 二、python3 property()

前面使用pure python equivalent描述property的内在机制，下面使用biud-in函数

### 1, read-only property

property()fset fdel并不是全部都需要的。 fget是必须的，缺失其他两个可以建立只读property, 这里将property()用做装饰器：任何对voltage property的修改和删除都会被拒绝。


```python
class Parrot:
    def __init__(self):
        self._voltage = 100000

    @property
    def voltage(self):
        return self._voltage

p1 = Parrot()
p1.voltage
```




    100000



### 2，增加对attribute的检查

当需要对attribute进行类型检查的时候:   
设置\_first_name默认值为'NAME',当调用setter时检查是否为字符类型，并阻止删除该属性


```python
class Person_a:
    def __init__(self):
        self._first_name = 'NAME'

    # Getter function
    @property
    def first_name(self):
        return self._first_name

    # Setter function
    @first_name.setter
    def first_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._first_name = value

    # Deleter function (optional)
    @first_name.deleter
    def first_name(self):
        raise AttributeError("Can't delete attribute")
```


```python
p1 = Person_a()
```


```python
p1.first_name
```




    'NAME'



如果想在实例初始化的时候就进行类型检查，对传入的参数进行检查：  
注意 \_\_init\_\_部分：


```python
class Person:
    def __init__(self, first_name):
        self.first_name = first_name

    # Getter function
    @property
    def first_name(self):
        return self._first_name

    # Setter function
    @first_name.setter
    def first_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._first_name = value

    # Deleter function (optional)
    @first_name.deleter
    def first_name(self):
        raise AttributeError("Can't delete attribute")

p = Person('Jimmy')
p.first_name
```




    'Jimmy'



通过设置 **self.first\_name** ，自动调用 setter 方法， 这个方法里面会进行参数的检查!!

property属性其实就是一系列相关绑定方法的集合。如果你去查看拥有property的类， 就会发现property本身的fget、fset和fdel属性就是类里面的普通方法: 


```python
Person.first_name.fget
```




    <function __main__.Person.first_name>



references:  
https://docs.python.org/3/howto/descriptor.html
https://docs.python.org/3/library/functions.html#property
http://python3-cookbook.readthedocs.io/zh_CN/latest/c08/p09_create_new_kind_of_class_or_instance_attribute.html


