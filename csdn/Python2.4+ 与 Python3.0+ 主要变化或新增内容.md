Python2.4+ 与 Python3.0+ 主要变化或新增内容 Python2                 Python3 print是内置命令
print变为函数 print >> f,x,y          print(x,y,file=f) print x,
print(x,end='') reload(M)               imp.reload(M) apply(f, ps, ks)
f(*ps, **ks) x <> y                  x != y long                    int 1234L
1234 d.has_key(k)            k in d 或 d.get(k) != None (has_key已死, in永生!!)
raw_input()             input() input()                 eval(input())
xrange(a,b)             range(a,b) file()                  open() x.next()
x.__next__() 且由next()方法调用 x.__getslice__()        x.__getitem__()
x.__setsilce__()        x.__setitem__() __cmp__()
删除了__cmp__(),改用__lt__(),__gt__(),__eq__()等 reduce()
functools.reduce() exefile(filename)       exec(open(filename).read()) 0567
0o567 (八进制)                         新增nonlocal关键字
str用于Unicode文本,bytes用于二进制文本                         新的迭代器方法range,map,zip等
新增集合解析与字典解析 u'unicodestr'           'unicodestr' raise E,V               raise
E(V) except E , x:           except E as x: file.xreadlines         for line
in file: (or X = iter(file)) d.keys(),d.items(),etc
list(d.keys()),list(d.items()),list(etc) map(),zip(),etc
list(map()),list(zip()),list(etc) x=d.keys(); x.sort()    sorted(d)
x.__nonzero__()         x.__bool__() x.__hex__,x.__bin__     x.__index__
types.ListType          list __metaclass__ = M       class C(metaclass = M):
__builtin__             builtins sys.exc_type,etc
sys.exc_info()[0],sys.exc_info()[1],... function.func_code
function.__code__                         增加Keyword-One参数
增加Ellipse对象                         简化了super()方法语法 用过-t,-tt控制缩进
混用空格与制表符视为错误 from M import *可以      只能出现在文件的顶层 出现在任何位置. class MyException:
class MyException(Exception): thread,Queue模块         改名_thread,queue
cPickle,SocketServer模块 改名_pickle,socketserver ConfigSparser模块
改名configsparser Tkinter模块              改名tkinter
其他模块整合到了如http模块,urllib, urllib2模块等 os.popen                subprocess.Popen
基于字符串的异常           基于类的异常                         新增类的property机制(类特性) 未绑定方法
都是函数 混合类型可比较排序         非数字混合类型比较发生错误 /是传统除法               取消了传统除法, /变为真除法
无函数注解                有函数注解 def f(a:100, b:str)->int 使用通过f.__annotation__
新增环境管理器with/as                         Python3.1支持多个环境管理器项 with A() as a, B()
as b                         扩展的序列解包 a, *b = seq
统一所有类为新式类                         增强__slot__类属性 if X: 优先X.__len__()
优先X.__bool__() type(I)区分类和类型       不再区分(不再区分新式类与经典类,同时扩展了元类) 静态方法需要self参数
静态方法根据声明直接使用 无异常链                  有异常链 raise exception from other_exception

