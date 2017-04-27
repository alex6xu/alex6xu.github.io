django包含了一个“信号分配器”使得当一些动作在框架的其他地方发生的时候，解耦的应用可以得到提醒。通俗来讲，就是一些动作发生的时候，信号允许特定的发送者
去提醒一些接受者，这是特别有用的设计因为有些代码对某些事件是特别感兴趣的，比如删除动作。
为此，django提供了很多内置的信号，比如一些常用的功能（以几个在django.db.models.signal目录下的信号为例）：
save：pre_save和post_savedelete：pre_delete和post_deletechange：m2m_changed
如果你想了解更多，可以查阅django的内置信号文档，本节的最后也会有一个所有内置信号的简略介绍。 监听信号
要想接受信号，你首先要注册一个接收器函数，当信号被Signal.connect()方法发射的时候，这个函数会被调用
Signal.connect(receiver[,sender=None,weak=True,dispatch_uid=None]) 参数解释： recei
ver：连接到这个信号的回调函数sender：信号的发送者weak：是否是弱引用，默认是真。因此，如果你的接收器是是一个本地函数，会被当做垃圾回收，如果你不
想，请在使用connect()方法的时候使用weak=Falsedispatch_uid：一个唯一的标识符给信号接收器，避免重复的信号被发送
下面让我们来看一个具体的例子来解释这些参数吧吧，这个例子以request_finished信号（每个HTTP请求结束的时候会被调用）：
回调函数receiver 回调函数可以是一个函数或者方法，比如我们可以这样定义一个接收器： def my_callback(sender,
**kwargs): print "Request finished!" 注意的是所有的信号处理器都需要这两个参数：sender和**kwargs。因为所有
的信号都是发送关键字参数的，可能你处理的时候没有任何参数，但不意味着在处理的过程中（在你写的处理函数之前）有任何的参数生成，如果没有传**kwargs参数的
话，可能会发生问题；基于这样的考虑，这两个参数都是必须的。 连接到你的receiver回调函数 有两种方法可以把信号和接收器连接到一起： connect方法
from django.core.signals import request_finished
request_finished.connect(my_callback) 装饰器方法 from django.core.signals import
request_finished from django.dispatch import receiver
@receiver(request_finished) def my_callback(sender, **kwargs): print "Request
finished!" 这样配置之后，每次HTTP接受的时候都会调用这个接收器回调函数了 绑定特定的发送者 记上面之后，你会不会想到这样的一个问题：每次都调用
，会不会很烦啊？如果是我的话，我肯定觉得很烦，毕竟我不是想接受所有人的信号的，所以你需要设置sender关键字参数
第二个知识：每一类的信号都对应着特定的发送者，所以要绑定发送者也得绑定对应的发送者类型，例如，request_finished对应的是handler
class，而pre_save对应则是model class 下面我们以pre-save为例子绑定特定的发送者（模型）： from
django.db.models.signals import pre_save from django.dispatch import receiver
from myapp.models import MyModel @receiver(pre_save, sender=MyModel) def
my_handler(sender, **kwargs): 预防重复的信号 使用dispatch_uid关键字参数
request_finished.connect(my_callback, dispatch_uid="my_unique_identifier")
说完了如何监听一个信号，下面我们继续讲解定义和发送信号吧】 定义信号 class Signal([providing_args=list])
所有的信号都是django.dispatch.Signal的实例，参数providing_args是一个信号提供给监听器的参数名的列表，比如： import
django.dispatch pizza_done =
django.dispatch.Signal(providing_args=["toppings", "size"])
这段代码定义了一个pizza_done的信号，参数有toppings和size 发送信号 有两个方法发送信号
Signal.send(sender,**kwargs) Signal.send_robust(sender,**kwargs)
sender参数是必须的，关键字参数可选 class PizzaStore(object): ... def send_pizza(self,
toppings, size): pizza_done.send(sender=self, toppings=toppings, size=size)
这两种方法都返回一个 元组对[(receiver,respose),...] 的 列表
，一个代表被调用的receiver回调函数和他们的response的列表 这两种方法的区别在于send不会捕捉任何的异常，（放任错误的传播），而send_r
obust则是捕捉所有的异常，并确保每个接收器都知道这个信号（发生错误了）（如果发生错误的话，错误实体和发生错误的接收器作为一个元组对一起返回给那个列表
断开信号 Signal.disconnect([receiver=None,sender=None,weak=True,dispatch_uid=None）
和监听信号类似 receiver参数用来指明那个接收器被断开，如果使用了dispatch_uid的话，receiver可以为None
总结，你可以使用django自带的信号，也可以自定义自己的信号，信号可以connect，可以send也可以disconnect等等

