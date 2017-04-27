若只使用python3.X, 下面可以不看了, 记住有个urllib的库就行了 python2.X 有这些库名可用: urllib, urllib2,
urllib3, httplib, httplib2, requests python3.X 有这些库名可用: urllib, urllib3,
httplib2, requests 两者都有的urllib3和requests, 它们不是标准库. urllib3
提供线程安全连接池和文件post支持,与urllib及urllib2的关系不大. requests 自称HTTP for Humans, 使用更简洁方便
对于python2.X: urllib和urllib2的主要区别:
urllib2可以接受Request对象为URL设置头信息,修改用户代理,设置cookie等,
urllib只能接受一个普通的URL.urllib提供一些比较原始基础的方法而urllib2没有这些, 比如 urlencode
urllib官方文档的几个例子 使用带参数的GET方法取回URL >>> import urllib >>> params =
urllib.urlencode({'spam': 1, 'eggs': 2, 'bacon': 0}) >>> f =
urllib.urlopen("http://www.musi-cal.com/cgi-bin/query?%s" % params) >>> print
f.read() 使用POST方法 >>> import urllib >>> params = urllib.urlencode({'spam': 1,
'eggs': 2, 'bacon': 0}) >>> f = urllib.urlopen("http://www.musi-cal.com/cgi-
bin/query", params) >>> print f.read() 使用HTTP代理,自动跟踪重定向 >>> import urllib >>>
proxies = {'http': 'http://proxy.example.com:8080/'} >>> opener =
urllib.FancyURLopener(proxies) >>> f = opener.open("http://www.python.org")
>>> f.read() 不使用代理 >>> import urllib >>> opener = urllib.FancyURLopener({})
>>> f = opener.open("http://www.python.org/") >>> f.read() urllib2的几个官方文档的例子:
GET一个URL >>> import urllib2 >>> f = urllib2.urlopen('http://www.python.org/')
>>> print f.read() 使用基本的HTTP认证 import urllib2 auth_handler =
urllib2.HTTPBasicAuthHandler() auth_handler.add_password(realm='PDQ
Application', uri='https://mahler:8092/site-updates.py', user='klem',
passwd='kadidd!ehopper') opener = urllib2.build_opener(auth_handler)
urllib2.install_opener(opener)
urllib2.urlopen('http://www.example.com/login.html') build_opener()
默认提供很多处理程序, 包括代理处理程序, 代理默认会被设置为环境变量所提供的. 一个使用代理的例子 proxy_handler =
urllib2.ProxyHandler({'http': 'http://www.example.com:3128/'})
proxy_auth_handler = urllib2.ProxyBasicAuthHandler()
proxy_auth_handler.add_password('realm', 'host', 'username', 'password')
opener = urllib2.build_opener(proxy_handler, proxy_auth_handler)
opener.open('http://www.example.com/login.html') 添加HTTP请求头部 import urllib2 req
= urllib2.Request('http://www.example.com/') req.add_header('Referer',
'http://www.python.org/') r = urllib2.urlopen(req) 更改User-agent import urllib2
opener = urllib2.build_opener() opener.addheaders = [('User-agent',
'Mozilla/5.0')] opener.open('http://www.example.com/') httplib 和 httplib2
httplib 是http客户端协议的实现,通常不直接使用, urllib是以httplib为基础 httplib2 是第三方库,
比httplib有更多特性 httplib比较底层，一般使用的话用urllib和urllib2即可 对于python3.X: 这里urllib成了一个包,
此包分成了几个模块,  urllib.request 用于打开和读取URL, urllib.error 用于处理前面request引起的异常,
urllib.parse 用于解析URL, urllib.robotparser用于解析robots.txt文件 python2.X 中的
urllib.urlopen()被废弃, urllib2.urlopen()相当于python3.X中的urllib.request.urlopen()
几个官方例子: GET一个URL >>> import urllib.request >>> with
urllib.request.urlopen('http://www.python.org/') as f: ... print(f.read(300))
PUT一个请求 import urllib.request DATA=b'some data' req =
urllib.request.Request(url='http://localhost:8080', data=DATA,method='PUT')
with urllib.request.urlopen(req) as f: pass print(f.status) print(f.reason)
基本的HTTP认证 import urllib.request auth_handler =
urllib.request.HTTPBasicAuthHandler() auth_handler.add_password(realm='PDQ
Application', uri='https://mahler:8092/site-updates.py', user='klem',
passwd='kadidd!ehopper') opener = urllib.request.build_opener(auth_handler)
urllib.request.install_opener(opener)
urllib.request.urlopen('http://www.example.com/login.html') 使用proxy
proxy_handler = urllib.request.ProxyHandler({'http':
'http://www.example.com:3128/'}) proxy_auth_handler =
urllib.request.ProxyBasicAuthHandler()
proxy_auth_handler.add_password('realm', 'host', 'username', 'password')
opener = urllib.request.build_opener(proxy_handler, proxy_auth_handler)
opener.open('http://www.example.com/login.html') 添加头部 import urllib.request
req = urllib.request.Request('http://www.example.com/')
req.add_header('Referer', 'http://www.python.org/') r =
urllib.request.urlopen(req) 更改User-agent import urllib.request opener =
urllib.request.build_opener() opener.addheaders = [('User-agent',
'Mozilla/5.0')] opener.open('http://www.example.com/') 使用GET时设置URL的参数 >>>
import urllib.request >>> import urllib.parse >>> params =
urllib.parse.urlencode({'spam': 1, 'eggs': 2, 'bacon': 0}) >>> url =
"http://www.musi-cal.com/cgi-bin/query?%s" % params >>> with
urllib.request.urlopen(url) as f: ... print(f.read().decode('utf-8')) ...
使用POST时设置参数 >>> import urllib.request >>> import urllib.parse >>> data =
urllib.parse.urlencode({'spam': 1, 'eggs': 2, 'bacon': 0}) >>> data =
data.encode('ascii') >>> with
urllib.request.urlopen("http://requestb.in/xrbl82xr", data) as f: ...
print(f.read().decode('utf-8')) ... 指定proxy >>> import urllib.request >>>
proxies = {'http': 'http://proxy.example.com:8080/'} >>> opener =
urllib.request.FancyURLopener(proxies) >>> with
opener.open("http://www.python.org") as f: ... f.read().decode('utf-8') ...
不使用proxy, 覆盖环境变量的proxy >>> import urllib.request >>> opener =
urllib.request.FancyURLopener({}) >>> with
opener.open("http://www.python.org/") as f: ... f.read().decode('utf-8') ...
python2.X中的httplib被重命名为 http.client 使用 2to3 工具转换源码时, 会自动处理这几个库的导入 总的来说,
使用python3, 记住只有urllib, 想要更简洁好用就用requests, 但不够通用  参考:
http://www.hacksparrow.com/python-difference-between-urllib-and-urllib2.html
http://blog.csdn.net/lxlzhn/article/details/10474281 http://www.codefrom.com/p
aper/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3urllib%E3%80%81urllib2%E5%8F%8Areques
ts http://www.cnblogs.com/wly923/archive/2013/05/07/3057122.html
http://stackoverflow.com/questions/2018026/should-i-use-urllib-urllib2-or-
requests http://stackoverflow.com/questions/3305250/python-urllib-vs-httplib
http://hustcalm.me/blog/2013/11/14/httplib-httplib2-urllib-urllib2-whats-the-
difference/

