[python] view plain copy   #!/usr/bin/python   #-*- coding: utf-8 -*-   '''''
Created on 2014-03-16    @author: Kris  '''   import urllib2, re, cookielib
def httpCrawler(url):       '''''      @summary: 网页抓取      '''       content =
httpRequest(url)       title = parseHtml(content)       saveData(title)
def httpRequest(url):       '''''      @summary: 网络请求      '''         try:
ret = None           SockFile = None           request = urllib2.Request(url)
request.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 6.0; Windows
NT 5.2; SV1; .NET CLR 1.1.4322)')           request.add_header('Pragma', 'no-
cache')           opener = urllib2.build_opener()           SockFile =
opener.open(request)           ret = SockFile.read()       finally:
if SockFile:               SockFile.close()                  return ret
def parseHtml(html):       '''''      @summary: 抓取结构化数据      '''       content
= None       pattern = '([^<]*?)'       temp = re.findall(pattern, html)
if temp:           content = temp[0]              return content          def
saveData(data):       '''''      @summary: 数据存储      '''       f =
open('test', 'wb')       f.write(data)       f.close()          if __name__ ==
'__main__':       url = 'http://www.baidu.com'       httpCrawler(url)

