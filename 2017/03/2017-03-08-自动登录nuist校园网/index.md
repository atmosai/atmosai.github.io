# 自动登录NUIST校园网


本文最初是为了开机自动登录某论坛进行签到所写，但为了防止扰乱论坛正常使用，仅介绍自动登录校园网脚本。

考虑到仅需要开机登录校园网，因此此处并未给出注销的代码。

```python
#-*- coding: utf-8 -*-
 
import requests
import base64
from os import getenv
 
username = getenv('IUSER')
password = getenv('PASSWORD')        
domain = getenv('DOMAIN')        
 
passwd = base64.b64encode(password.encode('utf-8')).decode()
 
login  = 'http://a.nuist.edu.cn/index.php/index/login'
 
headers = {'User-Agent' : 'Mozilla/5.0 (X11; Linux x86_64; rv:51.0) Gecko/20100101 Firefox/51.0',  
           'Referer' : 'http://a.nuist.edu.cn/'}  
 
loginData = {'domain' : domain,
            'enablemacauth' : 0,
            'password' : passwd,
            'username' : username
            }  
 
s = requests.session()
 
checkinfo = s.get('http://www.baidu.com')
if checkinfo.text.find(domain) > 0:
    getinfo = s.post(login, data = loginData, headers = headers)
 
    cookie = {'PHPSESSID' : s.cookies.values()[0], 
            'sunriseUsername' : username, 
            'sunriseDomain': domain, 
            'think_language': 'zh_CN'}
 
    print(getinfo.json().get('info'))
elif checkinfo.text.find('www.baidu.com'):    
    from datetime import datetime
 
    tiurl  = 'http://a.nuist.edu.cn/index.php/index/init?'
    tiinfo = s.get(tiurl)
    timearary = datetime.utcfromtimestamp(tiinfo.json().get('logout_timer'))
 
    print('网络已连接：%s.' % timearary.strftime('%H:%M:%S'))
else:
    print(checkinfo.text)
```

首先，加载 **requests**库，用于实现 **http 通信**，登录网页使用base64算法对密码进行了编码，加载**base64** 库用于对密码进行编码， 从 os 库中加载 **getenv** 获取系统**环境变量**。

```python
import requests
import base64
from os import getenv
```

使用程序时需要设置相应的环境变量。需要设置的环境变量为以下三行中的 IUSER, PASSWORD, DOMAIN 三个变量。

```python
username = getenv('IUSER')
password = getenv('PASSWORD')        
domain = getenv('DOMAIN')
```

有一点要注意：DOMAIN 可以为 NUIST，CMCC， ChinaNet， Unicom，分别对应 南京信息工程大学， 中国移动，中国电信， 中国联通。

以下分别为 登录页 URL，通信headers及登录时所需要的信息：

```python
login  = 'http://a.nuist.edu.cn/index.php/index/login'
 
headers = {'User-Agent' : 'Mozilla/5.0 (X11; Linux x86_64; rv:51.0) Gecko/20100101 Firefox/51.0',  
           'Referer' : 'http://a.nuist.edu.cn/'}  
 
loginData = {'domain' : domain,
            'enablemacauth' : 0,
            'password' : passwd,
            'username' : username
            }
```

建立 session，这里首先检查一下是否已经联网（百度除了可以被 ping 外，也可以检查是否联网）。如果已经联网了，将给出已经联网的时长。


```python
s = requests.session()
 
checkinfo = s.get('http://www.baidu.com')
if checkinfo.text.find(domain) > 0:
    getinfo = s.post(login, data = loginData, headers = headers)
 
    cookie = {'PHPSESSID' : s.cookies.values()[0], 
            'sunriseUsername' : username, 
            'sunriseDomain': domain, 
            'think_language': 'zh_CN'}
 
    print(getinfo.json().get('info'))
elif checkinfo.text.find('www.baidu.com'):    
    from datetime import datetime
 
    tiurl  = 'http://a.nuist.edu.cn/index.php/index/init?'
    tiinfo = s.get(tiurl)
    timearary = datetime.utcfromtimestamp(tiinfo.json().get('logout_timer'))
 
    print('网络已连接：%s.' % timearary.strftime('%H:%M:%S'))
else:
    print(checkinfo.text)
```

开机自动启动部分，不同系统设置不同。不再赘述。


