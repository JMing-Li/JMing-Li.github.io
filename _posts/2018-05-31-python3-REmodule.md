---
layout:     post
title:      "Python3 正则表达式"
subtitle:   "re模块"
date:       2018-05-31 17:00:00
author:     "JM"
header-img: "img/post-bg-simple.jpg"
tags:
    - Python3
    - 2018-05
---

## Python3 正则表达式 -- re模块

### 简单的正则表达式语法  

正则表达式可以实现一系列对字符串的操作，最常见的就是字符匹配。

#### 字符匹配

正则表达式中的绝大多数字符匹配字符本身，比如正则表达式text精确匹配字符串text 。（在case-insensitive 模式下这个还可以匹配TeXt等等）  

正则表达式中有其他一些字符是保留的元字符（metacharacter），它们不匹配本身，而是有特殊的用处，比如匹配特殊的位置（$匹配句尾），或者影响正则表达式的其他部分（比如A+匹配一个或多个A）。

metacharacter 完整列表如下：
>. ^ $ * + ? { } [ ] \ | ( )

**[ ] :**  

中括号[ ]用于指定一类想要匹配的字符的集合。  

[abcd]用于匹配字符a或b或c或d中的任何一个；也可以写成[a-d]。  

[ ]中的**metacharacter将不再起特殊的作用**，[abcd\$]用于匹配a,b,c,d或 $  

[^]用于匹配这个集合的补集, [^abc]用于匹配除了a，b，c之外的所有字符。



#### 字符的重复

指定正则表达式中某些部分重复的次数：  

例如ca\*t匹配字符ct,cat,caat等等。

metacharacter用法:  
![RE](https://raw.githubusercontent.com/JMing-Li/JMing-Li.github.io/master/img/in-post/RE.png)  

### 使用正则表达式

#### 编译正则表达式 

正则表达式首先要被编译成 **pattern object**:  

**re.compile(pattern, flags = 0)**  

正则表达式作为pattern参数传入函数，直接将正则表达式用string传入python虽然简介，但是会引起转义方面的问题，下面详解：

#### 使用raw string表示正则表达式 

python字符串中\是被当作转义字符的，而在正则表达式中，\也是元字符，输入的pattern首先它是python字符串，经过python转义；处理之后的字符串才作为正则表达式。使用raw string可以避免python的转义，这样输入的内容就直接作为正则表达式了！  

当想要匹配字符\时，regex需要2个\,而如果不使用raw string,需要4个\！


```python
import re
pat_obj = re.compile(r"[aeio]")
type(pat_obj)
```




    _sre.SRE_Pattern



#### 进行匹配

编译之后的正则表达式可以用来在文本中匹配，匹配使用的是**pattern object**的方法和属性。  

**pattern object**常见方法属性：  

|Method/Attribute  |Purpose |
|------------------|--------|
|match()        |在文本的开头匹配RE，返回匹配到的第一个结果|
|search()       |在文本的全部匹配RE，返回匹配到的**第一个**结果|
|findall()      |在文本的全部匹配RE，返回所有的结果组成的list|
|finditer()     |在文本的全部匹配RE，返回所有的结果的iterator|

match() search()如果没有匹配到，返回None， 如果匹配到，返回的结果将是一个**match object** 其中包含匹配到信息。  

**match object**常用方法属性：

|Method/Attribute |Purpose |
|-----------------|-------|
|group()  |Return the string matched by the RE|
|start()  |Return the starting position of the match |
|end()   |Return the ending position of the match |
|span()   |Return a tuple containing the (start, end) positions of the match |


```python
pat_obj = re.compile(r"[aeio]")
mch_obj = pat_obj.match("a line of text")
mch_obj
```




    <_sre.SRE_Match object; span=(0, 1), match='a'>




```python
mch_obj.group()
```




    'a'



匹配到的所有的string：


```python
pat_obj.findall("a line OF TEXT")
```




    ['a', 'i', 'e']



匹配到的所有的string的起始终止index：


```python
mch_obj = pat_obj.finditer("a line OF TEXT")
for i in mch_obj: 
    print(i.span())
```

    (0, 1)
    (3, 4)
    (5, 6)
    

### 模块级别函数

有时候并不需要先编译成pattern object, 再调用它的方法，模块级别函数可以直接使用match() serach(） findall()等函数，类似地，匹配失败返回None，匹配成功返回match object,etc.


```python
print(re.findall(r"[aeio]", "A line OF TEXT"))
```

    ['i', 'e']
    

### 编译的Flag

**compile(pattern, flag=0)**  

compile()函数flag参数指定编译时的模式，flag有全程和缩写；  

可以指定多个flag，使用 | 分割，常用的如下：  

|Flag       | Meaning|
|--------------|---------|
|re.DOTALL/re.S    |使得.可以匹配任何字符，包括换行符|
|re.IGNORECASE/re.I |大小写不敏感匹配|
|re.MULTILINE/re.M  |^ $ 的多行匹配|
|re.ASCII/re.A     |使得\w, \b, \s等只匹配ascii字符而非Unicode|  
|re.VERBOSE/re.X    |RE中的空格将被忽略，允许使用#注释 |

**re.MULTILINE** 多行模式:  

针对 ^ $ 这两个元字符，多行模式下可以匹配到多行。  

注意：多行匹配只是针对以上两个元字符，与match()，search()等方法无关，这些方法并非按行处理！  

**re.ASCII** ：

针对 \w \W \b \B \s \S 这几个元字符是否只匹配到ascii 字符。


```python
p = re.compile(r'''
\b   # first world border
\w+  # alphanumeric characters
\b   # second border
''', re.A | re.X)
p.findall("a text line\nthe 2nd line\n中文")
```




    ['a', 'text', 'line', 'the', '2nd', 'line']



### 更复杂的匹配

下面介绍更加复杂的匹配, 借助元字符，实现分组，取回文本的部分内容。

#### 更多的元字符

更多的元字符功能见开头的列表，^ $ \b 等称为 zero-width assertions，他们不消耗任何字符，不占宽度，所以**不能将它们设置成重复多个**， 例如\b匹配到单词的边界，显然可以插入无穷多的\b。

查找这些元字符本身，除了使用raw string并且转义，还可以利用[ ]。

#### 分组

很多时候不仅需要知道RE是否匹配，还需要对匹配到的部分进行解析，利用分组可以提取到匹配字符串的sub group.  

利用小括号（）对RE进行分组，匹配到的整体编号为0，各个subgroup根据左括号出现顺序依次编号。


```python
p = re.compile('(a(b)*c)d')
m = p.match('abbcd')
```

##### match object属性groups( )
列出所有的subgroup组成的tuple：  
注意是groups()不是group()


```python
m.groups()
```




    ('abbc', 'b')



上面列出的是两个subgroup,相应的编号为1，2，而整个RE匹配的是abbcd,对应group 0.

##### match object其他属性
其他的方法属性如 group( ), start( ),end( ), span( )都可以通过编号获取：


```python
print('0组: ' + m.group(0) + ' 1组: ' + m.group(1) + ' 2组: ' + m.group(2))
```

    0组: abbcd 1组: abbc 2组: b
    


```python
print('0组: ' + str(m.span(0)) + ' 1组: ' + str(m.span(1)) + ' 2组: ' + str(m.span(2)))
```

    0组: (0, 5) 1组: (0, 4) 2组: (2, 3)
    

group()返回字符串，可以接受多个组编号：  
span()返回tuple，start()end()返回数字，都只接受一个组编号


```python
m.group(1,2,1)
```




    ('abbc', 'b', 'abbc')



### 非捕获组 & 命名组

当RE变得复杂时，编号将变得困难，需要将组命名（捕获组）；  
当有些组只是用来标注在RE中的位置，而不像取出内容时，需要使用非捕获组


当捕获全部的组时,RE匹配到的部分span(0,5),有两个subgroup:

#### Non-capturing groups 


```python
m = re.search("([abc]*)(de)+", "abcde")
m.groups()
```




    ('abc', 'de')



使用  (?:&nbsp;)  来表明是non-capturing group：  
RE匹配到的部分span范围不变，只有一个subgroup


```python
m = re.search("(?:[abc]*)(de)+", "abcde")
m.groups()
```




    ('de',)



使用非捕获组时，除了不能取回该组的内容，其他操作与捕获组完全相同：非捕获组内可以添加任意RE，可以对组经行重复，可以将组放在其他的组（捕获或非捕获组）中。 

当在一个已有的RE中添加新的内容又不想改变既有的组的编号的时候，非捕获组很适合。

#### Named groups

python扩展的命名组使用 (?P&lt;name&gt;...) 来对subgroup命名：  

命名组既可以通过名字也可以通过组的编号来获取


```python
p = re.compile(r'a (?P<id>\b\w+\b)')
m = p.search('an apple a student')
m
```




    <_sre.SRE_Match object; span=(9, 18), match='a student'>




```python
m.group('id')
```




    'student'



#### Backreferences

使用组编号 (...)...\1代表编号为1的subgroup:  

或者使用组名字 (?P=name) ： 

以下两种RE等价


```python
p = re.compile(r'\b(\w+)\s+\1\b')
p = re.compile(r'\b(?P<id>\w+)\s+(?P=id)\b')
p.search('Paris in the the spring').group(0)
```




    'the the'



### Lookahead Assertions

Lookahead Assertions are **zero-width** assertions.  

**positive:  (?=...)** ...表示一小段RE  

一个pattern包含一个positive LA, 在文本的这个位置匹配...表示的RE，如果在这个位置RE匹配成功则LA匹配成功；RE匹配失败则LA匹配失败。但是一旦在这个位置尝试匹配这一小段RE，**matching engine不会advance**。pattern的剩余部分从LA的这个位置继续开始匹配。  

**negative:  (?!...)**  

注意：这不是分组！  

比较使用LA前后匹配到的string, 因为是zero-width,匹配LA时maching engine不会前进!


```python
re.search(r'.*sample\d.*[.]gz','old.sample3.fa.gz').group(0)
```




    'old.sample3.fa.gz'




```python
re.search(r'.*sample\d.*[.](?=gz)','old.sample3.fa.gz').group(0)
```




    'old.sample3.fa.'




```python
re.search(r'.*sample\d.*[.](?=gz).*','old.sample3.fa.gz').group(0)
```




    'old.sample3.fa.gz'



上面的例子，使用positive LA,当LA在pattern的最后的时候，因为maching engine不会前进了，所以匹配到的string不包含LA中的内容；  
当在LA (?=gz)后面加上.* 的时候，maching engine继续前进，相当于使用普通的RE。


假如有一串文件名，想筛选到fa或fastq文件，无论是否是压缩的：


```python
files = ['test.sample2.fastq.gz',  'sample2.config.gz',
         'old.sample3.fa.gz', 'sample1.fa','sample1.gtf.gz']

p = re.compile(r'''
.*
sample\d   # 文件名含有sample加上数字
[.]        # 相当于转义dot
(?!config)    # negative LA
(?=fa|fastq)  # positive LA
.*''', re.X)

for f in files:
    m = p.search(f)
    if m:
        print(m.group(0))
```

    test.sample2.fastq.gz
    old.sample3.fa.gz
    sample1.fa
    

假如有一串文件名，想剔除exe|cmd最后后缀的：


```python
files = ['11.txt','12.txt.gz','a.tar.gz','s.exe','b.cmd','cc.cmd.txt']
p = re.compile(r'.*[.](?!exe|cmd)[^.]*$')
for f in files:
    m = p.search(f)
    if m:
        print(m.group(0))
```

    11.txt
    12.txt.gz
    a.tar.gz
    cc.cmd.txt
    

#### 参考资料
https://docs.python.org/3/howto/regex.html#regex-howto  

https://songlee24.github.io/categories/Lang-Python/
