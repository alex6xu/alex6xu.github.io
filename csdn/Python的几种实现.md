下面分别列出几个主要的实现。 1.CPython：这是Python的官方版本，使用C语言实现，使用最为广泛，新的语言特性一般也最先出现在这里。
CPython实现会将源文件（py文件）转换成字节码文件（pyc文件），然后运行在Python虚拟机上。
2.Jython：这是Python的Java实现，相比于CPython，它与Java语言之间的互操作性要远远高于CPython和C语言之间的互操作性。
在Python中可以直接使用Java代码库，这使得使用Python可以方便地为Java程序写测试代码，更进一步，可以在Python中使用Swing等图形库编
写GUI程序。    Jython会将Python代码动态编译成Java字节码，然后在JVM上运行转换后的程序，这意味着此时Python程序与Java程序没
有区别，只是源代码不一样。    在Python 中写一个类，像使用Java 类一样使用这个类是很容易的事情。    你甚至可以把Jython
脚本静态地编译为Java 字节码。    示例代码： from java.lang import System
System.out.write('Hello World!\n') 3.Python for
.NET：它实质上是CPython实现的.NET托管版本，它与.NET库和程序代码有很好的互操作性。 4.IronPython：不同于Python for
.NET，它是Python的C#实现，并且它将Python代码编译成C#中间代码（与Jython类似），然后运行，它与.NET语言的互操作性也非常好。 5.
PyPy：Python的Python实现版本，原理是这样的，PyPy运行在CPython（或者其它实现）之上，用户程序运行在PyPy之上。它的一个目标是成为
Python语言自身的试验场，因为可以很容易地修改PyPy解释器的实现（因为它是使用Python写的）。
6.Stackless：CPython的一个局限就是每个Python函数调用都会产生一个C函数调用。 这意味着同时产生的函数调用是有限制的，因此Python
难以实现用户级的线程库和复杂递归应用。一旦超越这个限制，程序就会崩溃。Stackless的Python实现突破了这个限制，一个C栈帧可以拥有任意数量的Pyt
hon栈帧。这样你就能够拥有几乎无穷的函数调用，并能支持巨大数量的线程。Stackless唯一的问题就是它要对现有的CPython解释器做重大修改。所以它几
乎是一个独立的分支。另一个名为Greenlets的项目也支持微线程。它是一个标准的C扩展，因此不需要对标准Python解释器做任何修改。
下面的这篇文章对Stackless做了比较多的介绍，但是也比较难以读懂： 可爱的 Python：Python实现内幕

