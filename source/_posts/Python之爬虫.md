---
title: Python之爬虫
date: 2018-05-11 13:59:45
tags: Python
categories: Python
---



# BeautifulSoup

urlib是 Pyhton 的标准库（就是说你不需要额外安装就可以运行这个例子），包含了从网络请求数据，处理cookie，甚至改变像请求头和用户代理这些元数据的函数。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fq952y5gn6j318g0p0q88.jpg)

urlopen 用来打开并读取一个从网络获取的远程对象。因为它是一个非常通用的库（它可以轻松读取HTML文件、图像文件，或其他任何文件流），所以我们将在本书中频繁地使用它。

```python
# 在 urllib 库里面，查找 Python 的 request 模块，只导入一个urlopen函数
from urllib.request import urlopen
# 导入我们刚才安装的 BeautifulSoup对象
from bs4 import BeautifulSoup

html = urlopen("http://pythonscraping.com/pages/page1.html") # 打开url，获取html内容
bsObj = BeautifulSoup(html.read()) # 把页面内容传到BeautifulSoup
print(bsObj.h1) # 打印 html中.h1内容
```



## BeautifulSoup的常用方法

### .get_text()

**什么时候使用 get_text() 与什么时候应该保留标签？**

.get_text() 会把你正在处理的**HTML** 文档中所有的标签都清除，然后返回一个只包含文字的字符串。假如你正在处理一个包含许多超链接、段落和标签的大段源代码，那么.get_text() 会把这些超链接、段落和标签都清除掉，只剩下一串不带标签的文字。



用**BeautifulSoup** 对象查找你想要的信息，比直接在**HTML**文本里查找信息要简单的多。通常在你准备打印、存储和操作数据时，应该最后才使用.get_text()。一般情况下，你应该尽可能地保留**HTML**文档的标签结构。



### find（）和findAll（）

下面的两行代码是完全一样的;

```python
bsObj.findAll(id="text")
bsObj.findAll("",{"id":"text"})
```

### 后代标签和子标签

**子标签**：就是一个父标签的下一级

**后代标签**：父标签下面所有级别的标签

```python
for child in bsObj.find("table",{"id":"giftList"}).children:
    print(child)
```

上面的代码是获取子标签

如果用 descendants（）函数就是后代标签



