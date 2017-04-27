Python线程 Threading用于提供线程相关的操作，线程是应用程序中工作的最小单元。 1 2 3 4 5 6 7 8 9 10 11 12 13
14 #!/usr/bin/env python # -*- coding:utf-8 -*- import threading import time
def show(arg):     time.sleep(1)     print 'thread'+str(arg)    for i in
range(10):     t = threading.Thread(target=show, args=(i,))     t.start()
print 'main thread stop' 上述代码创建了10个“前台”线程，然后控制器就交给了CPU，CPU根据指定算法进行调度，分片执行指令。
更多方法： start            线程准备就绪，等待CPU调度setName      为线程设置名称getName
获取线程名称setDaemon   设置为后台线程或前台线程（默认）
如果是后台线程，主线程执行过程中，后台线程也在进行，主线程执行完毕后，后台线程不论成功与否，均停止
如果是前台线程，主线程执行过程中，前台线程也在进行，主线程执行完毕后，等待前台线程也执行完成后，程序停止join
逐个执行每个线程，执行完毕后继续往下执行，该方法使得多线程变得无意义run              线程被cpu调度后自动执行线程对象的run方法
自定义线程类 线程锁（Lock、RLock）
由于线程之间是进行随机调度，并且每个线程可能只执行n条执行之后，当多个线程同时修改同一条数据时可能会出现脏数据，所以，出现了线程锁 -
同一时刻允许一个线程执行操作。 未使用锁 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21
#!/usr/bin/env python #coding:utf-8     import threading import time
gl_num = 0     lock = threading.RLock()     def Func():     lock.acquire()
global gl_num     gl_num +=1     time.sleep(1)     print gl_num
lock.release()         for i in range(10):     t =
threading.Thread(target=Func)     t.start() 信号量（Semaphore） 互斥锁
同时只允许一个线程更改数据，而Semaphore是同时允许一定数量的线程更改数据
，比如厕所有3个坑，那最多只允许3个人上厕所，后面的人只能等里面有人出来了才能再进去。 1 2 3 4 5 6 7 8 9 10 11 12 13 14
15 import threading,time   def run(n):     semaphore.acquire()
time.sleep(1)     print("run the thread: %s" %n)     semaphore.release()   if
__name__ == '__main__':       num= 0     semaphore  =
threading.BoundedSemaphore(5) #最多允许5个线程同时运行     for i in range(20):         t
= threading.Thread(target=run,args=(i,))         t.start() 事件（event）
python线程的事件用于主线程控制其他线程的执行，事件主要提供了三个方法 set、wait、clear。
事件处理的机制：全局定义了一个“Flag”，如果“Flag”值为 False，那么当程序执行 event.wait
方法时就会阻塞，如果“Flag”值为True，那么event.wait 方法时便不再阻塞。
clear：将“Flag”设置为Falseset：将“Flag”设置为True 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16
17 18 19 20 21 #!/usr/bin/env python # -*- coding:utf-8 -*-   import threading
def do(event):     print 'start'     event.wait()     print 'execute'
event_obj = threading.Event() for i in range(10):     t =
threading.Thread(target=do, args=(event_obj,))     t.start()
event_obj.clear() inp = raw_input('input:') if inp == 'true':
event_obj.set() 条件（Condition） 使得线程等待，只有满足某条件时，才释放n个线程 1 2 3 4 5 6 7 8 9 10 11
12 13 14 15 16 17 18 19 20 21 22 import threading   def run(n):
con.acquire()     con.wait()     print("run the thread: %s" %n)
con.release()   if __name__ == '__main__':       con = threading.Condition()
for i in range(10):         t = threading.Thread(target=run, args=(i,))
t.start()       while True:         inp = input('>>>')         if inp == 'q':
break         con.acquire()         con.notify(int(inp))         con.release()
View Code Timer 定时器，指定n秒后执行某操作 1 2 3 4 5 6 7 8 from threading import Timer
def hello():     print("hello, world")   t = Timer(1, hello) t.start()  #
after 1 seconds, "hello, world" will be printed Python 进程 1 2 3 4 5 6 7 8 9 10
from multiprocessing import Process import threading import time    def
foo(i):     print 'say hi',i    for i in range(10):     p =
Process(target=foo,args=(i,))     p.start()
注意：由于进程之间的数据需要各自持有一份，所以创建进程需要的非常大的开销。 进程数据共享 进程各自持有一份数据，默认无法共享数据 进程间默认无法数据共享 1
2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27
#方法一，Array from multiprocessing import Process,Array temp = Array('i',
[11,22,33,44])   def Foo(i):     temp[i] = 100+i     for item in temp:
print i,'----->',item   for i in range(2):     p =
Process(target=Foo,args=(i,))     p.start()   #方法二：manage.dict()共享数据 from
multiprocessing import Process,Manager   manage = Manager() dic =
manage.dict()   def Foo(i):     dic[i] = 100+i     print dic.values()   for i
in range(2):     p = Process(target=Foo,args=(i,))     p.start()     p.join()
类型对应表 Code 当创建进程时（非使用时），共享数据会被拿到子进程中，当进程中执行完毕后，再赋值给原值。 进程锁实例 进程池
进程池内部维护一个进程序列，当使用时，则去进程池中获取一个进程，如果进程池序列中没有可供使用的进进程，那么程序就会等待，直到进程池中有可用进程为止。
进程池中有两个方法： applyapply_async 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
21 22 #!/usr/bin/env python # -*- coding:utf-8 -*- from  multiprocessing
import Process,Pool import time    def Foo(i):     time.sleep(2)     return
i+100    def Bar(arg):     print arg    pool = Pool(5) #print
pool.apply(Foo,(1,)) #print pool.apply_async(func =Foo, args=(1,)).get()
for i in range(10):     pool.apply_async(func=Foo, args=(i,),callback=Bar)
print 'end' pool.close() pool.join()#进程池中进程执行完毕后再关闭，如果注释，那么程序直接关闭。 协程
线程和进程的操作是由程序触发系统接口，最后的执行者是系统；协程的操作则是程序员。 协程存在的意义：对于多线程应用，CPU通过切片的方式来切换线程间的执行，线
程切换时需要耗时（保存状态，下次继续）。协程，则只使用一个线程，在一个线程中规定某个代码块执行顺序。
协程的适用场景：当程序中存在大量不需要CPU的操作时（IO），适用于协程； greenlet 1 2 3 4 5 6 7 8 9 10 11 12 13
14 15 16 17 18 19 20 21 22 #!/usr/bin/env python # -*- coding:utf-8 -*-
from greenlet import greenlet     def test1():     print 12     gr2.switch()
print 34     gr2.switch()     def test2():     print 56     gr1.switch()
print 78   gr1 = greenlet(test1) gr2 = greenlet(test2) gr1.switch() gevent 1 2
3 4 5 6 7 8 9 10 11 12 13 14 15 16 import gevent   def foo():
print('Running in foo')     gevent.sleep(0)     print('Explicit context switch
to foo again')   def bar():     print('Explicit context to bar')
gevent.sleep(0)     print('Implicit context switch back to bar')
gevent.joinall([     gevent.spawn(foo),     gevent.spawn(bar), ]) 遇到IO操作自动切换：
from gevent import monkey; monkey.patch_all() import gevent import urllib2 def
f(url): print('GET: %s' % url) resp = urllib2.urlopen(url) data = resp.read()
print('%d bytes received from %s.' % (len(data), url)) gevent.joinall([
gevent.spawn(f, 'https://www.python.org/'), gevent.spawn(f,
'https://www.yahoo.com/'), gevent.spawn(f, 'https://github.com/'), ])

