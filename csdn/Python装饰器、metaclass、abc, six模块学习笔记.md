　　最近接触到了Python中的decorator，metaclass，abc
Module，six.add_metaclass等内容，这里做一个简单的笔记。 　　主要资源： 　　1. PEP3119： Abstract Base
Classes 　　2. abc模块：abc Module，abc—Abstract Base Classes 　　3. metaclass:
“Python中metaclass解释”、浅析python的metaclass、PEP3115 　　4. 相关：Python 3 初探，第 2 部分:
高级主题 　　5. six.add_metaclass: six Module
装饰器的引入纯粹是一个“语法糖”，即让代码看起来更加易懂。装饰器引入前Python中已经存在了“class method”, "static
method"等包裹函数，不使用装饰器的结果是如果一个方法要被声明为class method，那么在他的“def”语句结束后需要立即使用"classmeth
od"将其注册成类方法。这样有一些弊端：当代码的读者开始读这个函数的时候，他一般看不到末尾的"classmethod"语句，所以可能直到看完整个函数的定义才
知道这是一个类方法，也即是最初没有装饰器时在定义的结尾对方法进行装饰的设定比较反人类；另外采用 method = classmethod(method)
方式写出来的代码，设计Python的大神们觉得 method 竟然重复出现了两次太多了，写这两次 method
的时间已经够他们喝一壶的了，所以引入了更为简洁的decorator。 　　装饰器以“@”标识，实质上是一层包裹函数，即函数的函数。写在函数定义（ def
语句）的前面，表示 def 语句后定义的函数受到装饰器的装饰，也就是说这个新定义的函数刚刚出生，立刻在函数定义结束的下一行代码运行装饰器给她穿点衣服遮羞。
metaclass是“类的类”，秉承Python“一切皆对象”的理念，Python中的类也是一类对象，metaclass的实例就是类（class），自己写m
etaclass时需要让其继承自type对象。关于metaclass的介绍，“主要资源”中相关的链接，不做赘述。
ABC（抽象基类），主要定义了基本类和最基本的抽象方法，可以为子类定义共有的API，不需要具体实现。 　　abc模块，Python
对于ABC的支持模块，定义了一个特殊的metaclass—— ABCMeta 还有一些装饰器—— @abstractmethod 和
@abstarctproperty 。 　　 abc.ABCMeta 是一个metaclass，用于在Python程序中创建抽象基类。 　　抽象基类可以不实
现具体的方法（当然也可以实现，只不过子类如果想调用抽象基类中定义的接口需要使用super()）而是将其留给派生类实现。抽象基类可以被子类直接继承，也可以将其
他的类”注册“（register）到其门下当虚拟子类，虚拟子类的好处是你实现的第三方子类不需要直接继承自基类但是仍然可以声称自己子类中的方法实现了基类规定的
接口（issubclass(), issubinstance()）！ 　　虚拟子类是通过调用metaclass是 abc.ABCMeta 的抽象基类的
register 方法注册到抽象基类门下的，可以实现抽象基类中的部分API接口，也可以根本不实现，但是issubclass(),
issubinstance()进行判断时仍然返回真值。 　　直接继承抽象基类的子类就没有这么灵活，在metaclass是 abc.ABCMeta的抽象基类中
可以声明”抽象方法“和“抽象属性”，直接继承自抽象基类的子类虽然判断issubclass()时为真，但只有完全覆写（实现）了抽象基类中的“抽象”内容后，才能
被实例化，而通过注册的虚拟子类则不受此影响。 　　metaclass为 abc.ABCMeta
的抽象基类如果想要声明“抽象方法”，可以使用abc模块中的装饰器 @abstractmethod ，如果想声明“抽象属性”，可以使用abc模块中的
@abstractproperty 。 　　最后，为什么要提six模块呢，six模块是Python为了兼容Python 2.x 和Python
3.x提供的一个模块，该模块中有一个针对类的装饰器 @six.add_metaclass(MetaClass)
可以为两个版本的Python类方便地添加metaclass。这样我们就可以同时利用Python中的abc模块和six模块在类的定义前添加
@six.add_metaclass(abc.ABCMeta) 来优雅地声明一个抽象基础类了！ 　　从理论层面打通了，下面上代码，首先看一下类装饰器
@six.add_metaclass(MetaClass) 的用法，在下面的代码中，我们希望声明类 MyClass 的metaclass是类 Meta
，注意类 Meta 需要是一个metaclass。 import six @six.add_metaclass(Meta) class
MyClass(object): pass 在Python 3 等价于 import six class MyClass(object, metaclass
= Meta): pass 在Python 2.x (x >= 6)中等价于 import six class MyClass(object):
__metaclass__ = Meta pass 或者直接用引入装饰器的目的： import six class MyClass(object):
pass MyClass = six.add_metaclass(Meta)(MyClass) 　　类装饰器
@six.add_metaclass(MetaClass)
的作用是在不同版本的Python之间提供一个优雅的声明类的metaclass的手段，事实上不用它也可以，只是使用了它代码更为整洁明了。
下面结合一个特殊的metaclass即 abc.ABCMeta 来看一段代码： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16
17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42
43 44 45 46 47 48 49 50 51 import abc import six
@six.add_metaclass(abc.ABCMeta) class PluginBase(object):
@abc.abstractmethod     def func_a(self,data):         """         an abstract
method need to be implemented         """     @abc.abstractmethod     def
func_b(self,output, data):         """         another abstract method need to
be implemented         """           class RegisteredImplementation(object):
def func_c(self, data):         print "Method in third-party class, "+
str(data)       class SubclassImplementation(PluginBase):           def
func_a(self,data):         print "Overriding func_a, "+ str(data)
def func_b(self,data):         print "Overriding func_b, "+ str(data)
def func_d(self, data):         print data
PluginBase.register(RegisteredImplementation)   if __name__=='__main__':
for sc in PluginBase.__subclasses__():         print "subclass of PluginBase:
" + sc.__name__     print("")     print issubclass(RegisteredImplementation,
PluginBase)     print isinstance(RegisteredImplementation(), PluginBase)
print issubclass(SubclassImplementation, PluginBase)     print("")     obj1 =
RegisteredImplementation()     obj1.func_c("It's right!")     print("")
obj2 = SubclassImplementation()     obj2.func_a("It's right!")     print ""
上面这端代码的含义是： 　　声明一个metaclass是 abc.ABCMeta 的抽象基类 PluginBase
，为其定义两个抽象方法，等待派生类的实现。接着定义了一个第三方类 RegisterdImplementation ，将其注册为类 PluginBase
的虚拟子类。再定义一个子类 SubclassImplementation 直接继承自抽象基类 PluginBase 。 　　接着进行试验，运行结果如下：
subclass of PluginBase: SubclassImplementation True True True Method in third-
party class, It's right! Overriding func_a, It's right!  　　从运行的结果我们可以看出：
虚拟子类不算做直接继承子类，因此可以不实现抽象基类 PluginBase 的任何方法；但直接继承的子类 SubclassImplementation
必须完全实现抽象基类的抽象方法才能够实例化（这里可以注释掉 26 - 30 行的代码实验）。
同时，不论是虚拟子类还是直接继承子类，issubclass()和issubinstance()判断他们与抽象基类的关系时都返回真值。

