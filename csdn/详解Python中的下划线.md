这篇文章说两个问题： 问一：下划线变量 单下划线开头的变量，Pyhthon规定为内部变量（私有变量），from M import *
时，这种变量并不会导入进来，例如： foo.py #foo.py bar = 10 bar2 = 11 _bar = 20 __bar = 30
foo2.py #foo2.py from foo import * if __name__ == '__main__': print locals()
执行 python foo2.py，输出： {'bar2': 11, 'bar': 10, '...省略'} 输入结果中并没有 _bar和__bar，因为它
们都是以下划线开头的变量，所以没有导入进来，但是如果你非要把这些变量导入进来也是可以的，使用import时，明确导入具体的变量时就行了。如：
#foo2.py from foo import * from foo import _bar from foo import __bar if
__name__ == '__main__': print locals() 输出： {'_bar': 20, 'bar2': 11, 'bar': 10,
'__bar': 30, ‘...省略'} 单下划线结尾的变量：用于避免于Python关键字冲突的变量，如class_：
Tkinter.Toplevel(master, class_='ClassName') 如上所说的变量讲的是定义在模块中的变量，属于模块中的属性，如果这些
变量定义在函数里面，那它和普通的变量没什么两样的，都是局部变量。此外，单下划线同样适用于函数。 双下划线开头的变量：它在模块中还是当作单下划线看待，但出现在
类中作为类属性就不一样了，在运行时该类属性会被“混淆"，不能直接访问，需要在该变量前加上下划线和类名才能访问。如： class Foo(object):
boo = 40 _boo = 50 __boo = 60 # _Foo__boo def __init__(self): self.__booo = 70
def __test(self): #_Foo__test print "__test" if __name__ == '__main__': print
Foo.boo print Foo._boo print Foo._Foo__boo foo = Foo() print foo._Foo__booo
foo._Foo__test() 这样可以防止与父类或子类中同名的__xxx属性发生冲突。
开始和结尾都有的双下划线的变量：此类变量属于魔法对象，如：init，file，你永远不要自己也发明个出来。 问二：all
all对象是装有字符串的列表对象，他会覆盖 from import * 的默认行为：如 #foo.py __all__ = ['bar', 'baz']
waz = 5 bar = 10 def baz(): return 'baz' from foo import * print bar print baz
# 异常 print waz 在foo.py里面定义了all后，import 就会按照 all定义的内容导入，所以这里 print
waz就抛异常了，因为它不在 all里面。为外，你可以把下划线开头的变量的字符串形式加入到all中，这样 import 也能看到这些变量。

