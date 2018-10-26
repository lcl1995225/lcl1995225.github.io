title: Python爬虫笔记（二）_urllib+selenium利用cookie
author: Lcl
tags: []
categories:
  - Python
date: 2018-10-26 03:36:00
---
Python爬虫笔记（二）_urllib+selenium利用cookie

备注：主要内容来自互联网，经过修改与整理使得更加符合LCL的阅读习惯。
Lcl的习惯：章节号##，副标题###，其他事项####

本教程可能分为以下几个部分：
第一部分：使用urllib发送和处理简单请求
第二部分：urllib+selenium利用cookie
第三部分：beautifulSoup与正则表达式
第四部分：高级爬虫（分布式与框架初探)
<!-- more -->
## 第四章
### 使用headers与proxy代理

#### 核心代码
```python
import urllib.request

if __name__ == "__main__":
   #访问网址
   urls = ['http://httpbin.org/get','https://httpbin.org/get']

   #这是代理IP，http与https有各自的ip
   proxy = {
           'http':'109.207.59.70:30668',
           'https':'107.150.122.73:3128'
           }
   #创建ProxyHandler
   proxy_handler = urllib.request.ProxyHandler(proxy)
   #创建Opener
   opener = urllib.request.build_opener(proxy_handler)
   #定义User Agent
   header = {
       'User-Agent':'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36'
           }
   
   for url in urls:
       #在请求头中加入user-agent
       req = urllib.request.Request(url,headers=header)
       #使用自定义的Opener打开请求
       response = opener.open(req,timeout=10)
       #读取相应信息并解码
       html = response.read().decode("utf-8")
       #打印信息
       print(html)

'''输出
{
 "args": {},
 "headers": {
   "Accept-Encoding": "identity",
   "Connection": "close",
   "Host": "httpbin.org",
   "User-Agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
 },
 "origin": "109.207.59.70",
 "url": "http://httpbin.org/get"
}

{
 "args": {},
 "headers": {
   "Accept-Encoding": "identity",
   "Connection": "close",
   "Host": "httpbin.org",
   "User-Agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
 },
 "origin": "107.150.122.73",
 "url": "https://httpbin.org/get"
}



'''
```

加入header的时候，建议使用urllib.request.Request()对象

加入headers并使用代理
建议使用这个网站获取浏览器请求和响应头的关键信息（http://httpbin.org/get 注：这个网站也有https版本，将http改成https即可，访问此网站主页，可以获得更多测试工具，如IP等 ）


### 新内容解释：
#### urllib.request.Request()
构造一个完整的请求，加入header的时候，建议传入headers={}字典。而不是创建后再req.add_header

class urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
```python
data = urllib.parse.urlencode(formData(字典对象)).encode('utf-8')
headers = {} #字典对象
req = urllib.request.Request(url,data,headers）
req.add_header('key','value')
```
#### 测试时使用的网站
1. http://httpbin.org/get
建议使用这个网站获取浏览器请求和响应头的关键信息
注：这个网站也有https版本，将http改成https即可，访问此网站主页，可以获得很多测试工具。
2. 在远程或本地搭建[DWVA](http://www.dvwa.co.uk/)

### 使用代理访问并设置header的步骤总结如下
```python
#构造request
req = urllib.request.Request(url,data,headers={})
#→
opener = urllib.request.build_opener(urllib.request.ProxyHandler(为http与https建立的代理dict))
#→
reasponse = opener.open(req)
html = response.read().decode("相应的编码")
```

#### 附录：常见的user-agent

1. Android
Mozilla/5.0 (Linux; Android 4.1.1; Nexus 7 Build/JRO03D) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.166 Safari/535.19
Mozilla/5.0 (Linux; U; Android 4.0.4; en-gb; GT-I9300 Build/IMM76D) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30
Mozilla/5.0 (Linux; U; Android 2.2; en-gb; GT-P1000 Build/FROYO) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1

2. Firefox
Mozilla/5.0 (Windows NT 6.2; WOW64; rv:21.0) Gecko/20100101 Firefox/21.0
Mozilla/5.0 (Android; Mobile; rv:14.0) Gecko/14.0 Firefox/14.0

3. Google Chrome
Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/27.0.1453.94 Safari/537.36
Mozilla/5.0 (Linux; Android 4.0.4; Galaxy Nexus Build/IMM76B) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.133 Mobile Safari/535.19

4. iOS
Mozilla/5.0 (iPad; CPU OS 5_0 like Mac OS X) AppleWebKit/534.46 (KHTML, like Gecko) Version/5.1 Mobile/9A334 Safari/7534.48.3
Mozilla/5.0 (iPod; U; CPU like Mac OS X; en) AppleWebKit/420.1 (KHTML, like Gecko) Version/3.0 Mobile/3A101a Safari/419.3


## 第五章
### 利用cookie
现有的网站登陆普遍需要cookie。而现有的网站验证机制比较复杂，不妨在云服务器上搭建dwva环境供自己测试。
### 核心代码：
```python
import urllib.request
import urllib.parse
from http import cookiejar
import re
if __name__ == '__main__':
   #声明一个CookieJar对象实例来保存cookie
   cookie = cookiejar.CookieJar()
   #利用urllib.request库的HTTPCookieProcessor对象来创建cookie处理器,也就是CookieHandler
   handler= urllib.request.HTTPCookieProcessor(cookie)
   #通过CookieHandler创建opener
   opener = urllib.request.build_opener(handler)
   #此处的open方法打开网页
   url = 'http://118.25.194.91/dvwa-master/login.php'
   #尝试获取csrf-token
   response = opener.open(url).read().decode('utf-8')
   str = re.findall(r"name='user_token' value='(.*?)'",response)

   data = {
           'username':'admin',
           'password':'password',
           'Login':'Login',
           'user_token':str[0],
           }
   data = urllib.parse.urlencode(data).encode('utf-8')
   req = urllib.request.Request(url,data=data)
   response = opener.open(req).read().decode('utf-8')

   print(response)
```
### 新内容解释：
#### opener与cookie的关系
获取的cookie默认与 打开此网页的opener关联，所以需要使用获取cookie的opener打开需要登录的网页


## 第六章
### 同时使用proxy与cookie（多个handler）

```python
import urllib.request
import urllib.parse
from http import cookiejar
import re
if __name__ == '__main__':
   
   #创建ProxyHandler
   proxy = {
           'http':'195.78.101.185:48942',
         }
   proxyHandler = urllib.request.ProxyHandler(proxy)
   
   #创建CookieHandler
   #声明一个CookieJar对象实例来保存cookie
   cookie = cookiejar.CookieJar()
   cookieHandler= urllib.request.HTTPCookieProcessor(cookie)
   
   #创建同时包括ProxyHandler与CookieHandler的opener
   opener = urllib.request.build_opener(cookieHandler,proxyHandler)

   # 获取csrf-token进行模拟登陆
   url = 'http://118.25.194.91/dvwa-master/login.php'
   response = opener.open(url,timeout=10).read().decode('utf-8')
   str = re.findall(r"name='user_token' value='(.*?)'",response)

	# 进行模拟登陆，登陆后cookie会通过cookieHandler传回去
   data = {
           'username':'admin',
           'password':'password',
           'Login':'Login',
           'user_token':str[0],
           }
   data = urllib.parse.urlencode(data).encode('utf-8')
   req = urllib.request.Request(url,data=data)
   response = opener.open(req,timeout=10).read().decode('utf-8')

   # 访问一个可以显示当前IP的页面
   url = 'http://118.25.194.91/dvwa-master/vulnerabilities/fi/?page=file3.php'
   response = opener.open(url,timeout=10).read().decode('utf-8')
   str = re.findall(r"Your IP address is:(.*?)<h2>More info</h2>",response,re.DOTALL)
   req = urllib.request.Request(url)
   response = opener.open(req,timeout=10).read().decode('utf-8')

   print(str)
```

### 新内容解释：
可以使用多个handler创建一个opener
opener = urllib.request.build_opener(cookieHandler,proxyHandler)


## 第七章
### 使用selenium获取cookie并继续工作

#### 1、仅使用selenium
使用selenium，使用chrome的headless模式静默模拟登陆获取cookie，关闭后重新使用selenium直接获取cookie进行模拟登陆

```python
from selenium import webdriver
import time

#使用headless模式打开浏览器
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument('--headless')

#声明浏览器使用的驱动，并打开一个空白的浏览器
driver = webdriver.Chrome("chromedriver.exe",options=chrome_options)

#使用打开的chrome打开地址
loginurl = 'http://118.25.194.91/dvwa-master/index.php'
driver.get(loginurl)
time.sleep(1)

#往表单中填充信息并点击登录
username = driver.find_element_by_name("username")
username.send_keys('admin')
password = driver.find_element_by_name("password")
password.send_keys('password')
login = driver.find_element_by_name("Login")
login.click()
time.sleep(1)
print(driver.current_url)
#获得登录后的cookie
list_cookies = driver.get_cookies()
cookies = []
driver.quit()
time.sleep(1)

#在上文关闭浏览器后，重新打开浏览器，不使用headless模式
driver2 = webdriver.Chrome("chromedriver.exe")
#必须要先打开一次等待登录的界面，（此时当然是未登录的）才可以完成后续的add_cookie，不然会报错
driver2.get(loginurl)
for cookie in list_cookies:
   driver2.add_cookie(cookie)
#重新打开需要登录的界面，发现已经登录了
driver2.get(loginurl)
```

#### 2 使用selenium+手工，完成百度的模拟登陆
研究的问题：
使用selenium打开浏览器后，python端暂停，人工完成登陆，selenium是否可以获得cookie并用于下次的模拟登陆？
经验证，可以，验证方式如下↓


```python
from selenium import webdriver
import time
import json
#使用headless模式打开浏览器
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
#使用隐身模式，隔绝外部影响
chrome_options.add_argument('--incognito')
#不加载图片
#prefs = {"profile.managed_default_content_settings.images": 2}
#chrome_options.add_experimental_option("prefs", prefs)

#声明浏览器使用的驱动，并打开一个空白的浏览器
driver = webdriver.Chrome("chromedriver.exe",options=chrome_options)

#使用打开的chrome打开地址
loginurl = 'https://www.baidu.com/'
driver.get(loginurl)
cookie_before = driver.get_cookies()
if input("自己完成模拟登陆后，请输入小写的ok：") == "ok":
   print("已经保存登陆后的cookie，即将关闭并重新打开浏览器并载入cookie")
   cookie_after = driver.get_cookies()
   driver.quit()
   time.sleep(1)
   driver = webdriver.Chrome("chromedriver.exe",options=chrome_options)
   driver.get(loginurl)
   for cookie in cookie_after:
       driver.add_cookie(cookie)
   cookie_after_addin = driver.get_cookies()
   driver.get(loginurl)
```

### 新内容解释：
#### json与str的转换
list或者dict→→json.dumps(list)→str→→json.loads(str)→→list或者dict
助记：
json.loads，载入json状的字符串str供计算机使用，比如str→dict
而json.dumps 转储，将计算机使用的格式转储为json状字符串str，比如json→dict

#### selenium常用配置（使用隐身模式并不加载图片）

from selenium.webdriver.chrome.options import Options
chrome_options = Options()
#使用隐身模式，隔绝外部影响
chrome_options.add_argument('--incognito')
#不加载图片
prefs = {"profile.managed_default_content_settings.images": 2}
chrome_options.add_experimental_option("prefs", prefs)


#### driver.add_cookie的原理
以百度cookie的修改方式举例
新增原来不存在的字段，若新增name-value，将 自动补全其他字段，如expiry等。

driver.add_cookie({'name':"test","value":"value-test"})
{'domain': 'www.baidu.com', 'expiry': 2170944786, 'httpOnly': False, 'name': 'test', 'path': '/', 'secure': True, 'value': 'value-test'}

若修改原来已经存在的字段，也会增添新字段。除name-value字段外，其他将被自动补全。若再次新增同name字段，将仅修改后添加的而不新增。
{'domain': '.www.baidu.com', 'expiry': 2486310942, 'httpOnly': False, 'name': 'ORIGIN', 'path': '/', 'secure': True, 'value': '2'}
#↓
driver.add_cookie({'name':"ORIGIN","value":"value-test"})


## 第八章
### selenium进行复杂登录并配合urllib工作

#### 了解LWPCookie的标准格式

```python
以LWPCookie举例，首先使用标准库生成一个标准的LWP-COOKIE格式

import urllib.request
import urllib.parse
from http import cookiejar
import re
if __name__ == '__main__':
    cookie = cookiejar.LWPCookieJar()
    handler= urllib.request.HTTPCookieProcessor(cookie)
    opener = urllib.request.build_opener(handler)
    url = 'http://118.25.194.91/dvwa-master/login.php'
    response = opener.open(url).read().decode('utf-8')
    str = re.findall(r"name='user_token' value='(.*?)'",response)
    data = {
            'username':'admin',
            'password':'password',
            'Login':'Login',
            'user_token':str[0],
            }
    data = urllib.parse.urlencode(data).encode('utf-8')
    req = urllib.request.Request(url,data=data)
    response = opener.open(req).read().decode('utf-8')
    cookie.save('realLWP.txt',True,True)
```
```
realLWP.txt
#LWP-Cookies-2.0
Set-Cookie3: PHPSESSID=oetigbp2v5esgdped1vved2ho1; path="/"; domain="118.25.194.91"; path_spec; discard; httponly=None; version=0
Set-Cookie3: security=impossible; path="/dvwa-master"; domain="118.25.194.91"; discard; httponly=None; version=0
```
经测试，部分字段可以删除（如path_spec;等），调整为最简格式如下，格式说明：
第一行必须为#LWP-Cookies-2.0
第二行开始，开头必须为name=value，且不加引号;domain与path顺序可以替换，最后必须为version=0，最后是否加分号;都不会报错。
```
#LWP-Cookies-2.0
Set-Cookie3:PHPSESSID=oetigbp2v5esgdped1vved2ho1;domain="118.25.194.91";path="/";version=0
Set-Cookie3:security=impossible;path="/dvwa-master";domain="118.25.194.91";version=0
```

#### 将selenium获得的cookie与LWPCookie的标准格式进行转换,并成功模拟淘宝网登录
driver.get_cookies()
seleium-cookie（已经从list转为str）
```
[{"domain": "118.25.194.91", "httpOnly": true, "name": "security", "path": "/dvwa-master", "secure": false, "value": "impossible"}, 
{"domain": "118.25.194.91", "httpOnly": true, "name": "PHPSESSID", "path": "/", "secure": false, "value": "nuoncm7tu449fa66t766282am2"}]
```
#LWPCookie最简
```
#LWP-Cookies-2.0
Set-Cookie3:security=impossible;domain="118.25.194.91";path="/dvwa-master";version=0
Set-Cookie3:PHPSESSID=nuoncm7tu449fa66t766282am2;domain="118.25.194.91";path="/";version=0
```

附上代码

```python
import urllib.request
import urllib.parse
import urllib.error
import http.cookiejar
from selenium import webdriver
import json

#使用headless模式打开浏览器
from selenium.webdriver.chrome.options import Options
chrome_options = Options()

#使用隐身模式，隔绝外部影响
chrome_options.add_argument('--incognito')

#声明浏览器使用的驱动，并打开一个空白的浏览器
driver = webdriver.Chrome("chromedriver.exe",options=chrome_options)

#使用打开的chrome打开地址
loginurl = 'https://member1.taobao.com/member/fresh/account_security.htm'
driver.get(loginurl)
if input("自行完成登陆后，请输入小写的ok：") == "ok":
    print("已经保存登陆后的cookie，即将关闭浏览器并使用urllib完成后续工作")
    cookie_after = driver.get_cookies()
    driver.quit()
    with open("selCookie.txt",mode='w') as f:
        f.write(json.dumps(cookie_after))


##         使用urllib继续后续工作
        
# 定义一个转换cookie格式的函数
def selCookie2LWP(selCookie,LWPCookie):
    import json
    with open(selCookie) as f:
        selCookieStr = f.read()
    selCookieList = json.loads(selCookieStr)
    cookieList = ['Set-Cookie3:{}={};domain="{}";path="{}";version=0\r\n'.format(item["name"],item["value"],item["domain"],item["path"]) for item in selCookieList]
    LWPCookieStr = '#LWP-Cookies-2.0\r\n'
    for cookie in cookieList:
        LWPCookieStr+=cookie
    with open(LWPCookie,mode='w') as f:
        f.write(LWPCookieStr)

selCookie = "selCookie.txt"
LWPCookie = "LWPCookie.txt"
# 调用函数完成cookie的格式转换
selCookie2LWP(selCookie,LWPCookie)
cookie = http.cookiejar.LWPCookieJar(LWPCookie)
cookie.load(LWPCookie, ignore_discard=True, ignore_expires=True)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)

get_url = loginurl  # 利用cookie请求访问另一个网址
get_request = urllib.request.Request(get_url)
get_response = opener.open(get_request)
html = get_response.read().decode('gb2312',errors='ignore')
print(html)
# 既然已经能灵活运用cookie了，就可以为所欲为了。
```
