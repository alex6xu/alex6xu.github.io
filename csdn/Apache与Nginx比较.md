Nginx的高并发得益于其采用了epoll模型，与传统的服务器程序架构不同，epoll是linux内核2.6以后才出现的。下面通过比较Apache和Ngin
x工作原理来比较。         传统Apache都是多进程或者多线程来工作，假设是多进程工作（prefork），apache会先生成几个进程，类似进程池
的工作原理，只不过这里的进程池会随着请求数目的增加而增加。对于每一个连接，apache都是在一个进程内处理完毕。具体是 recv（），以及根据 URI
去进行磁盘I/O来寻找文件，还有 send（）都是阻塞的。其实说白了都是 apche 对于套接字的I/O，读或者写，但是读或者写都是阻塞的，阻塞意味着进程就
得挂起进入sleep状态，那么一旦连接数很多，Apache必然要生成更多的进程来响应请求，一旦进程多了，CPU对于进程的切换就频繁了，很耗资源和时间，所以就
导致apache性能下降了，说白了就是处理不过来这么多进程了。其实仔细想想，如果对于进程每个请求都没有阻塞，那么效率肯定会提高很多。
Nginx采用epoll模型，异步非阻塞。对于Nginx来说，把一个完整的连接请求处理都划分成了事件，一个一个的事件。比如accept（）， recv（），
磁盘I/O，send（）等，每部分都有相应的模块去处理，一个完整的请求可能是由几百个模块去处理。真正核心的就是事件收集和分发模块，这就是管理所有模块的核心。
只有核心模块的调度才能让对应的模块占用CPU资源，从而处理请求。拿一个HTTP请求来说，首先在事件收集分发模块注册感兴趣的监听事件，注册好之后不阻塞直接返回
，接下来就不需要再管了，等待有连接来了内核会通知你(epoll的轮询会告诉进程)，cpu就可以处理其他事情去了。一旦有请求来，那么对整个请求分配相应的上下文
（其实已经预先分配好），这时候再注册新的感兴趣的事件(read函数)，同样客户端数据来了内核会自动通知进程可以去读数据了，读了数据之后就是解析，解析完后去磁
盘找资源（I/O），一旦I/O完成会通知进程，进程开始给客户端发回数据send()，这时候也不是阻塞的，调用后就等内核发回通知发送的结果就行。整个下来把一个
请求分成了很多个阶段，每个阶段都到很多模块去注册，然后处理，都是异步非阻塞。异步这里指的就是做一个事情，不需要等返回结果，做好了会自动通知你。
select/epoll的特点 select的特点：select 选择句柄的时候，是遍历所有句柄，也就是说句柄有事件响应时，select需要遍历所有句柄才能
获取到哪些句柄有事件通知，因此效率是非常低。但是如果连接很少的情况下， select和epoll的LT触发模式相比， 性能上差别不大。
这里要多说一句，select支持的句柄数是有限制的， 同时只支持1024个，这个是句柄集合限制的，如果超过这个限制，很可能导致溢出，而且非常不容易发现问题，
当然可以通过修改linux的socket内核调整这个参数。 epoll的特点：epoll对于句柄事件的选择不是遍历的，是事件响应的，就是句柄上事件来就马上选
择出来，不需要遍历整个句柄链表，因此效率非常高，内核将句柄用红黑树保存的。
对于epoll而言还有ET和LT的区别，LT表示水平触发，ET表示边缘触发，两者在性能以及代码实现上差别也是非常大的。
Epoll模型主要负责对大量并发用户的请求进行及时处理，完成服务器与客户端的数据交互。其具体的实现步骤如下： (a)
使用epoll_create()函数创建文件描述，设定将可管理的最大socket描述符数目。 (b)
创建与epoll关联的接收线程，应用程序可以创建多个接收线程来处理epoll上的读通知事件，线程的数量依赖于程序的具体需要。 (c)
创建一个侦听socket描述符ListenSock；将该描述符设定为非阻塞模式，调用Listen（）函数在套接字上侦听有无新的连接请求，在
epoll_event结构中设置要处理的事件类型EPOLLIN，工作方式为
epoll_ET，以提高工作效率，同时使用epoll_ctl()注册事件，最后启动网络监视线程。 (d)
网络监视线程启动循环，epoll_wait()等待epoll事件发生。 (e)
如果epoll事件表明有新的连接请求，则调用accept（）函数，将用户socket描述符添加到epoll_data联合体，同时设定该描述符为非
阻塞，并在epoll_event结构中设置要处理的事件类型为读和写，工作方式为epoll_ET. (f)
如果epoll事件表明socket描述符上有数据可读，则将该socket描述符加入可读队列，通知接收线程读入数据，并将接收到的数据放入到接收数据
的链表中，经逻辑处理后，将反馈的数据包放入到发送数据链表中，等待由发送线程发送。 epoll的操作就这么简单，总共不过4个
API：epoll_create, epoll_ctl, epoll_wait和close。       可以举一个简单的例子来说明Apache的工作流程，
我们平时去餐厅吃饭。餐厅的工作模式是一个服务员全程服务客户，流程是这样，服务员在门口等候客人(listen)，客人到了就接待安排的餐桌上(accept)，等
着客户点菜(request uri)，去厨房叫师傅下单做菜（磁盘I/O），等待厨房做好（read），然后给客人上菜(send)，整个下来服务员(进程)很多地
方是阻塞的。这样客人一多（HTTP请求一多），餐厅只能通过叫更多的服务员来服务（fork进程），但是由于餐厅资源是有限的（CPU），一旦服务员太多管理成本很
高（CPU上下文切换），这样就进入一个瓶颈。 再来看看Nginx得怎么处理？餐厅门口挂个门铃（注册epoll模型的listen），一旦有客人（HTTP请求）
到达，派一个服务员去接待（accept），之后服务员就去忙其他事情了（比如再去接待客人），等这位客人点好餐就叫服务员（数据到了read()），服务员过来拿走
菜单到厨房（磁盘I/O），服务员又做其他事情去了，等厨房做好了菜也喊服务员（磁盘I/O结束），服务员再给客人上菜（send()），厨房做好一个菜就给客人上一
个，中间服务员可以去干其他事情。整个过程被切分成很多个阶段，每个阶段都有相应的服务模块。我们想想，这样一旦客人多了，餐厅也能招待更多的人。         
不管是Nginx还是Squid这种反向代理，其网络模式都是事件驱动。事件驱动其实是很老的技术，早期的select、poll都是如此。后来基于内核通知的更高级
事件机制出现，如libevent里的epoll，使事件驱动性能得以提高。事件驱动的本质还是IO事件，应用程序在多个IO句柄间快速切换，实现所谓的异步IO。事
件驱动服务器，最适合做的就是这种IO密集型工作，如反向代理，它在客户端与WEB服务器之间起一个数据中转作用，纯粹是IO操作，自身并不涉及到复杂计算。反向代理
用事件驱动来做，显然更好，一个工作进程就可以run了，没有进程、线程管理的开销，CPU、内存消耗都小。
所以Nginx、Squid都是这样做的。当然，Nginx也可以是多进程 + 事件驱动的模式，几个进程跑libevent，不需要Apache那样动辄数百的进程
数。Nginx处理静态文件效果也很好，那是因为静态文件本身也是磁盘IO操作，处理过程一样。至于说多少万的并发连接，这个毫无意义。随手写个网络程序都能处理几万
的并发，但如果大部分客户端阻塞在那里，就没什么价值。 　　再看看Apache或者Resin这类应用服务器，之所以称他们为应用服务器，是因为他们真的要跑具体的
业务应用，如科学计算、图形图像、数据库读写等。它们很可能是CPU密集型的服务，事件驱动并不合适。例如一个计算耗时2秒，那么这2秒就是完全阻塞的，什么even
t都没用。想想MySQL如果改成事件驱动会怎么样，一个大型的join或sort就会阻塞住所有客户端。这个时候多进程或线程就体现出优势，每个进程各干各的事，互
不阻塞和干扰。当然，现代CPU越来越快，单个计算阻塞的时间可能很小，但只要有阻塞，事件编程就毫无优势。所以进程、线程这类技术，并不会消失，而是与事件机制相辅
相成，长期存在。 　　总言之，事件驱动适合于IO密集型服务，多进程或线程适合于CPU密集型服务，它们各有各的优势，并不存在谁取代谁的倾向。

