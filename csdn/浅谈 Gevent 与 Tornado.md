回到顶部 从 Tornado 说起 刚开始，对 Tornado 的感觉最为新鲜，在官网介绍里其是一个无阻塞的Web服务器以及相关工具的集合，但
个人更为倾向其为一个颇为完备的微型 web 框架。Tornado 性能好的关键是其无阻塞异步的特性，但这魔术
似的效果是如何达成的呢？迷思与困惑。我那小脑袋里的思维还停留于多进程（多线程）那样的并发模型中， 实在有点难以理解 Tornado 的异步机制。
通过查阅各式文章以及源代码，整体的框架脉络开始逐渐在脑海中显现出来。其实，Tornado 的异步模型
是由事件驱动以及特定的回调函数（callback）所组成的！一直没有弄明白，Tornado 具体是如何实现
无阻塞异步，当清楚了事件驱动和回调函数的概念后，事情似乎又变得简单起来了。 对于一般的程序，在执行阶段若遇到 I/O 事件，整个进程将被阻塞住，直到 I/O
事件结束，程序又继续执行。 接设我们对一些 I/O 事件进行了定制，使其可以立即返回（即无阻塞），那么程序将能立即继续执行。但 问题又来了，那当 I/O
事件完成后又该怎么办呢？此时，回调函数的威力就出来了，我只需要将进行特定 处理的回调函数与该 I/O 事件绑定起来，当该 I/O
事件完成后就调用绑定的回调函数，就可以处理具体的 I/O 事件啦。啊，似乎还有一个问题，回调函数要如何与 I/O 事件绑定起来？最简单的想法是，直接通过
一个 while True 循环不断的轮询，当检测到 I/O 事件完成了即触发回调函数。但是，这样的效率当然不会 高，利用系统中高效的 I/O
事件轮询机制（epoll on Linux, kqueue on most BSD）就是最明智的 解决方案。于是，无阻塞 I/O
+事件驱动+高效轮询方式便组成了 Tornado 的异步模型。 Tornado 的核心是 ioloop 和 iostream
这两个模块，前者提供了一个高效的 I/O 事件循环，后者则封装了 一个无阻塞的 socket 。通过向 ioloop 中添加网络 I/O 事件，利用无阻塞的
socket ，再搭配相应的回调 函数，便可达到梦寐以求的高效异步执行啦。多说无益，来看一下具体的示例： from tornado import
ioloop from tornado.httpclient import AsyncHTTPClient urls =
['http://www.google.com', 'http://www.yandex.ru', 'http://www.python.org'] def
print_head(response): print ('%s: %s bytes: %r' % (response.request.url,
len(response.body), response.body[:50])) http_client = AsyncHTTPClient() for
url in urls: print ('Starting %s' % url) http_client.fetch(url, print_head)
ioloop.IOLoop.instance().start() 因为使用了 AsyncHTTPClient
来处理请求操作，整个示例是异步执行的，即三个url请求无等待的依次发出。 我们可以看到 fetch 方法使用了 print_head
函数来作为回调函数，这意味着，当 fetch 完成了请求操作， 相应的 print_head 函数便会被触发调用。恩，... 额，...，乍看起来，使用
Tornado 进行异步编程似乎 并不难，让人跃跃欲试。但实际上，在现实生活中，事件驱动的编程还是会很费脑力，需要一定的创造性思维。 不过，这也许是
Tornado 受欢迎的原因之一呢。 :)

