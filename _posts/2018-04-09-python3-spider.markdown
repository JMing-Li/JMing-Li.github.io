---
layout:     post
title:      "python3 spider by API"
subtitle:   ""
date:       2018-04-09
author:     "JM"
header-img: "img/post-bg-horse.jpg"
catalog: true
tags:
    - Python3 spider
    - 2018-04
---

## 爬虫：

###爬一个电影的评分：摔跤吧爸爸 为例

import 两个模块


```python
import requests, json
```

向豆瓣API发送请求，得到response对象


```python
url_visit = 'https://api.douban.com/v2/movie/26387939'
resp = requests.get(url_visit)
```


```python
type(resp)
```




    requests.models.Response



resp是一个response对象，这个对象包含一些属性，比如它的编码方式：


```python
resp.encoding
```




    'utf-8'


resp的text属性，content属性，都是包含了a JSON document

```python
resp.content?
```

对resp.content进行json解析,得到一个字典，命名为movie_info


```python
movie_info = json.loads(resp.content)
```


```python
movie_info
```




    {'alt': 'https://movie.douban.com/movie/26387939',
     'alt_title': '摔跤吧！爸爸 / 我和我的冠军女儿(台)',
     'attrs': {'cast': ['阿米尔·汗 Aamir Khan',
       '法缇玛·萨那·纱卡 Fatima Sana Shaikh',
       '桑亚·玛荷塔 Sanya Malhotra',
       '阿帕尔夏克提·库拉那 Aparshakti Khurana',
       '沙克希·坦沃 Sakshi Tanwar',
       '泽伊拉·沃西姆 Zaira Wasim',
       '苏哈妮·巴特纳格尔 Suhani Bhatnagar',
       '里特维克·萨霍里 Ritwik Sahore',
       '吉里什·库卡尼 Girish Kulkarni'],
      'country': ['印度'],
      'director': ['涅提·蒂瓦里 Nitesh Tiwari'],
      'language': ['印地语'],
      'movie_duration': ['161分钟(印度)', '140分钟(中国大陆)'],
      'movie_type': ['剧情', '传记', '运动', '家庭'],
      'pubdate': ['2016-12-23(印度)', '2017-05-05(中国大陆)'],
      'title': ['Dangal'],
      'writer': ['比于什·古普塔 Piyush Gupta',
       '施热亚·简 Shreyas Jain',
       '尼基尔·麦罗特拉 Nikhil Mehrotra',
       '涅提·蒂瓦里 Nitesh Tiwari'],
      'year': ['2016']},
     'author': [{'name': '涅提·蒂瓦里 Nitesh Tiwari'}],
     'id': 'https://api.douban.com/movie/26387939',
     'image': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2457983084.jpg',
     'mobile_link': 'https://m.douban.com/movie/subject/26387939/',
     'rating': {'average': '9.1', 'max': 10, 'min': 0, 'numRaters': 501315},
     'summary': '马哈维亚（阿米尔·汗 Aamir Khan 饰）曾经是一名前途无量的摔跤运动员，在放弃了职业生涯后，他最大的遗憾就是没有能够替国家赢得金牌。马哈维亚将这份希望寄托在了尚未出生的儿子身上，哪知道妻子接连给他生了两个女儿，取名吉塔（法缇玛·萨那·纱卡 Fatima Sana Shaikh 饰）和巴比塔（桑亚·玛荷塔 Sanya Malhotra 饰）。让马哈维亚没有想到的是，两个姑娘展现出了杰出的摔跤天赋，让他幡然醒悟，就算是女孩，也能够昂首挺胸的站在比赛场上，为了国家和她们自己赢得荣誉。\n就这样，在马哈维亚的指导下，吉塔和巴比塔开始了艰苦的训练，两人进步神速，很快就因为在比赛中连连获胜而成为了当地的名人。为了获得更多的机会，吉塔进入了国家体育学院学习，在那里，她将面对更大的诱惑和更多的选择。',
     'tags': [{'count': 110002, 'name': '励志'},
      {'count': 100961, 'name': '印度'},
      {'count': 67954, 'name': '成长'},
      {'count': 63945, 'name': '亲情'},
      {'count': 51133, 'name': '女权'},
      {'count': 50856, 'name': '体育'},
      {'count': 47816, 'name': '女性'},
      {'count': 44149, 'name': '运动'}],
     'title': 'Dangal'}




```python
movie_rating = movie_info['rating']['average']
```

提取电影名字，主演的第一个


```python
movie_name = movie_info['alt_title']
```


```python
movie_star = movie_info['attrs']['cast'][0] 
```

将需要的信息连接成一个字符串方便写入文件


```python
info = movie_name + '\t' + movie_rating+ '\t' + movie_star 
print(info)
```

    摔跤吧！爸爸 / 我和我的冠军女儿(台)	9.1	阿米尔·汗 Aamir Khan
    


```python
info
```




    '摔跤吧！爸爸 / 我和我的冠军女儿(台)\t9.1\t阿米尔·汗 Aamir Khan'




```python
import os
os.getcwd()
```




    'C:\\Users\\jm\\ZYL'



将字符串写入文件中


```python
with open('file1.txt','w') as f1:
    f1.write(info)
```

打开刚才的文件看看写的内容


```python
f = open('file1.txt','r')
```


```python
f.read()  # 读入所有内容
```




    '摔跤吧！爸爸 / 我和我的冠军女儿(台)\t9.1\t阿米尔·汗 Aamir Khan'




```python
f.close() #看完关闭文件
```


## 多个电影


```python
id_list=[26387939,11803087,20451290]
```

尝试一下循环


```python
for i in id_list: 
    print(i)
    url_visit = 'https://api.douban.com/v2/movie/search?q={}'.format(i)
    print(url_visit)
```

    26387939
    https://api.douban.com/v2/movie/search?q=26387939
    11803087
    https://api.douban.com/v2/movie/search?q=11803087
    20451290
    https://api.douban.com/v2/movie/search?q=20451290
    


```python
with open('file2.txt','w') as f:
    for i in id_list:
        url_visit = 'https://api.douban.com/v2/movie/{}'.format(i)
        resp = requests.get(url_visit)
        movie_info = json.loads(resp.content)
        movie_name = movie_info['alt_title']
        movie_rating = movie_info['rating']['average']
        movie_star = movie_info['attrs']['cast'][0]
        info = 'name: {} \t rating: {} \t star: {} \n'.format(movie_name,movie_rating,movie_star)
        f.write(info)
```


```python
f=open('file2.txt','r')
line = f.readlines()
for i in line: print(i)
f.close()
```

    name: 摔跤吧！爸爸 / 我和我的冠军女儿(台) 	 rating: 9.1 	 star: 阿米尔·汗 Aamir Khan 
    
    name: 异形：契约 / 异形：圣约(港/台) 	 rating: 7.3 	 star: 迈克尔·法斯宾德 Michael Fassbender 
    
    name: 新木乃伊 / 盗墓迷城(港) 	 rating: 4.8 	 star: 汤姆·克鲁斯 Tom Cruise 
    
    
