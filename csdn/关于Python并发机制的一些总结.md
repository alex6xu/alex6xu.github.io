经过一番研究，发现Python跟并发相关的东西，做个记录： 1，版本相关 Python2中并发大多使用第三方lib实现，thread, threading,
greenlet, eventlet, stackless,processing..........等，使用时需要先import。
Python3中开始有yield, 是Python3中新增的功能，称之为generator（生成器），个人理解用来实现协程的功能。

