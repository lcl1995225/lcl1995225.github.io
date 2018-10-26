title: Python爬虫笔记（一）_使用urllib发送和处理简单请求
author: Lcl
tags: []
categories:
  - Python
date: 2018-10-25 21:07:00
---
备注：主要内容来自互联网，经过修改与整理使得更加符合LCL的阅读习惯。
Lcl的习惯：章节号##，副标题###，其他事项####

本教程可能分为以下几个部分：
第一部分：使用urllib发送和处理简单请求
第二部分：urllib+selenium利用cookie
第三部分：beautifulSoup与正则表达式
第四部分：高级爬虫（分布式与框架初探)
<!-- more -->
## 第一章 
### 利用urllib进行简单的网页抓取
#### 核心代码
```Python
from urllib import request
import chardet
response = request.urlopen("https://www.163.com/")
html = response.read()#读取出来的是bytes
charset = chardet.detect(html)
html = html.decode(charset['encoding'],errors='ignore')
```

#### 新内容解释
##### 1、chardet：
为第三方模块，可以检测byte的编码，若使用Anaconda，则已经自带，该模块可以用来自动判断网页编码的类型。
该模块在实际使用时偶尔会碰到问题，即无法检测部分网页的编码，如[腾讯主页](https://www.qq.com)
##### 2、什么时候使用encode，什么时候使用decode?
str--->(encode，编码)--->bytes（二进制）
bytes（二进制）--->(decode，解码)--->str
助记：将字符编码为计算机理解的二进制语言。
举例：
str(小明abc）--->str.encode('utf-8',errors='ignore')--->bytes（b'\xe5\xb0\x8f\xe6\x98\x8eabc' 小明abc）
bytes（b'\xe5\xb0\x8f\xe6\x98\x8eabc' 小明abc）--->str.decode('utf-8',errors='ignore')--->str(小明abc）

#### 练习：文件编码转换
写法1：
使用python的with open语句完成文件编码转换，需要注意的是，如果f.read()运行完后指针会到最后，如果需要重复读取可以将其赋值为中间变量。
```python
from urllib import request
import chardet
with open('cookie.txt',encoding='gb2312',errors='ignore',newline='\r\n') as f:
	a = f.read()
   with open('utf-8.txt',mode='wb') as g:
       g.write(a.encode('utf-8'))
```
写法2：（读取写入均为字节，在中间转码。推荐使用这种方式，因为这种方式不需要处理换行问题。）：
```python
from urllib import request
import chardet
with open('cookie.txt',mode = 'rb') as f:
   a = f.read()
   with open('utf-8.txt',mode='wb') as g:
       g.write(a.decode(chardet.detect(a)['encoding']).encode('utf-8'))
```

## 第二章
### 使用post发送数据
#### 核心代码：
```python
import urllib
import json

requestURL = 'http://fanyi.youdao.com/translate'
formData = {
    'i':'我爱你',
    'from':'AUTO',
    'to':'AUTO',
    'smartresult':'dict',
    'client':'fanyideskweb',
    'salt':1539952880862,
    'sign':'3e6b698c9b9863f602a0eaab4cbdb567',
    'doctype':'json',
    'version':2.1,
    'keyfrom':'fanyi.web',
    'action':'FY_BY_CLICKBUTTION',
    'typoResult':'false'
}

#urlencode方法将请求的data转换成URL编码的String转换标准格式，encode将string转换为bytes
data = urllib.parse.urlencode(formData).encode('utf-8')
response = urllib.request.urlopen(requestURL,data)#传递完格式的数据
html = response.read().decode('utf-8')#读取信息并解码
translate_results = json.loads(html)#str→dict
print("翻译的结果是：{}".format(translate_results))
```

#### 新内容解释：

##### urllib.request.urlopen( )
主要作用是使用默认的opener打开网页，建议设置timeout限制其最大读取时间
urllib.request.urlopen(url, data=None[, timeout ], cafile=None, capath=None, cadefault=False,
context=None)
Open the URL url, which can be either a string or a Request object.
data must be an object specifying additional data to be sent to the server, or None if no such data is
needed.
##### urllib.parse.urlencode(可传入dict).encode('utf-8')
主要作用是将dict转换为url编码后的string，再将string转换为bytes供下一步调用，例如
```python
formData = {
  'i':'我爱你',  
  'from': 'AUTO',
}
data = urllib.parse.urlencode(formData).encode('utf-8')
b'i=%E6%88%91%E7%88%B1%E4%BD%A0&from=AUTO'
#如果不加 str.encode('utf-8')，则返回
'i=%E6%88%91%E7%88%B1%E4%BD%A0&from=AUTO'
```
Convert a mapping object or a sequence of two-element tuples, which may contain str or bytes
objects, to a percent-encoded ASCII text string. If the resultant string is to be used as a data for
POST operation with the urlopen() function, then it should be encoded to bytes, otherwise it would
result in a TypeError.

## 第三章
### error异常处理
#### 核心代码：
```python
import urllib

if __name__ == "__main__":
   url = "http://www.baidu.com/lcl.html"
   req = urllib.request.Request(url)
   try:
       responese = urllib.request.urlopen(req)
   except urllib.error.HTTPError as e:
       print(e.code)
   except urllib.error.URLError as e:
       print(e)
   else:
       print('如果try中的部分成功了，运行这个')
   finally:
       print('无论是否成功，均运行这里')
```


### 新内容解释：
#### try...except...except...else...finally
```python
try:
    print('先尝试运行这里')
except urllib.error.HTTPError as e:
    print('再尝试运行这里')
except urllib.error.URLError as e:
    print('然后尝试运行这里')
else:
   print('如果try中的部分成功了，运行这个')
finally:
   print('无论是否成功，均运行这里')
```

#### URLError与HTTPError的关系
urllib.error.URLError（处理此模块的所有异常）→衍生到子类→urllib.error.HTTPError（仅处理存在错误代码的HTTP出错信息，例如404(找不到页面)）。
如果想用HTTPError和URLError一起捕获异常，那么需要将HTTPError放在URLError的前面，因为HTTPError是URLError的一个子类。







>本部分主要参考资料：
1. [Jack_Gui的CSDN博客](https://blog.csdn.net/c406495762/article/details/58716886)
2. Python library reference_3.6.4