semget()      可以使用系统调用semget()创建一个新的信号量集，或者存取一个已经存在的信号量集：  系统调用：semget();
原型：intsemget(key_t key,int nsems,int semflg);
返回值：如果成功，则返回信号量集的IPC标识符。如果失败，则返回-1：errno=EACCESS(没有权限) EEXIST(信号量集已经存在，无法创建)
EIDRM(信号量集已经删除) ENOENT(信号量集不存在，同时没有使用IPC_CREAT) ENOMEM(没有足够的内存创建新的信号量集)
ENOSPC(超出限制)     系统调用semget()的第一个参数是关键字值（一般是由系统调用ftok()返回的）。系统内核将此值和系统中存在的其他的信
号量集的关键字值进行比较。打开和存取操作与参数semflg中的内容相关。IPC_CREAT如果信号量集在系统内核中不存在，则创建信号量集。IPC_EXCL当
和 IPC_CREAT一同使用时，如果信号量集已经存在，则调用失败。如果单独使用IPC_CREAT，则semget()要么返回新创建的信号量集的标识符，要么
返回系统中已经存在的同样的关键字值的信号量的标识符。如果IPC_EXCL和IPC_CREAT一同使用，则要么返回新创建的信号量集的标识符，要么返回-1。IP
C_EXCL单独使用没有意义。参数nsems指出了一个新的信号量集中应该创建的信号量的个数。信号量集中最多的信号量的个数是在linux/sem.h中定义的：
#defineSEMMSL32/*<=512maxnumofsemaphoresperid*/ 下面是一个打开和创建信号量集的程序：
intopen_semaphore_set(key_t keyval,int numsems) { intsid; if(!numsems)
return(-1); if((sid=semget(mykey,numsems,IPC_CREAT|0660))==-1) { return(-1); }
return(sid); } };
============================================================== semop()
系统调用：semop(); 调用原型：int semop(int semid,struct sembuf*sops,unsign ednsops);
返回值：0，如果成功。-1，如果失败：errno=E2BIG(nsops大于最大的ops数目) EACCESS(权限不够)
EAGAIN(使用了IPC_NOWAIT，但操作不能继续进行) EFAULT(sops指向的地址无效) EIDRM(信号量集已经删除)
EINTR(当睡眠时接收到其他信号) EINVAL(信号量集不存在,或者semid无效)
ENOMEM(使用了SEM_UNDO,但无足够的内存创建所需的数据结构) ERANGE(信号量值超出范围)      第一个参数是关键字值。第二个参数是指向
将要操作的数组的指针。第三个参数是数组中的操作的个数。参数sops指向由sembuf组成的数组。此数组是在linux/sem.h中定义的： /*semop
systemcall takes an array of these*/ structsembuf{ ushortsem_num;/*semaphore
index in array*/ shortsem_op;/*semaphore operation*/ shortsem_flg;/*operation
flags*/ sem_num将要处理的信号量的个数。 sem_op要执行的操作。 sem_flg操作标志。     如果sem_op是负数，那么信号量将减
去它的值。这和信号量控制的资源有关。如果没有使用IPC_NOWAIT，那么调用进程将进入睡眠状态，直到信号量控制的资源可以使用为止。如果sem_op是正数，
则信号量加上它的值。这也就是进程释放信号量控制的资源。最后，如果sem_op是0，那么调用进程将调用sleep()，直到信号量的值为0。这在一个进程等待完全
空闲的资源时使用。 ===============================================================
semctl() 系统调用：semctl(); 原型：int semctl(int semid,int semnum,int cmd,union
semunarg); 返回值：如果成功，则为一个正数。 如果失败，则为-1：errno=EACCESS(权限不够) EFAULT(arg指向的地址无效)
EIDRM(信号量集已经删除) EINVAL(信号量集不存在，或者semid无效) EPERM(EUID没有cmd的权利) ERANGE(信号量值超出范围)
系统调用semctl用来执行在信号量集上的控制操作。这和在消息队列中的系统调用msgctl是十分相似的。但这两个系统调用的参数略有不同。因为信号量一般是作为
一个信号量集使用的，而不是一个单独的信号量。所以在信号量集的操作中，不但要知道IPC关键字值，也要知道信号量集中的具体的信号量。这两个系统调用都使用了参数c
md，它用来指出要操作的具体命令。两个系统调用中的最后一个参数也不一样。在系统调用msgctl中，最后一个参数是指向内核中使用的数据结构的指针。我们使用此数
据结构来取得有关消息队列的一些信息，以及设置或者改变队列的存取权限和使用者。但在信号量中支持额外的可选的命令，这样就要求有一个更为复杂的数据结构。
系统调用semctl()的第一个参数是关键字值。第二个参数是信号量数目。     参数cmd中可以使用的命令如下：
·IPC_STAT读取一个信号量集的数据结构semid_ds，并将其存储在semun中的buf参数中。
·IPC_SET设置信号量集的数据结构semid_ds中的元素ipc_perm，其值取自semun中的buf参数。
·IPC_RMID将信号量集从内存中删除。     ·GETALL用于读取信号量集中的所有信号量的值。     ·GETNCNT返回正在等待资源的进程数目。
·GETPID返回最后一个执行semop操作的进程的PID。     ·GETVAL返回信号量集中的一个单个的信号量的值。
·GETZCNT返回这在等待完全空闲的资源的进程数目。     ·SETALL设置信号量集中的所有的信号量的值。
·SETVAL设置信号量集中的一个单独的信号量的值。     参数arg代表一个semun的实例。semun是在linux/sem.h中定义的： /*arg
for semctl systemcalls.*/ unionsemun{ intval;/*value for SETVAL*/
structsemid_ds*buf;/*buffer for IPC_STAT&IPC;_SET*/ ushort*array;/*array for
GETALL&SETALL;*/ structseminfo*__buf;/*buffer for IPC_INFO*/ void*__pad;     v
al当执行SETVAL命令时使用。buf在IPC_STAT/IPC_SET命令中使用。代表了内核中使用的信号量的数据结构。array在使用GETALL/SE
TALL命令时使用的指针。     下面的程序返回信号量的值。当使用GETVAL命令时，调用中的最后一个参数被忽略：
intget_sem_val(intsid,intsemnum) { return(semctl(sid,semnum,GETVAL,0)); }
下面是一个实际应用的例子： #defineMAX_PRINTERS5 printer_usage() { int x;
for(x=0;x<MAX_PRINTERS;x++) printf("Printer%d:%d/n/r",x,get_sem_val(sid,x)); }
下面的程序可以用来初始化一个新的信号量值： void init_semaphore(int sid,int semnum,int initval) {
union semunsemopts; semopts.val=initval; semctl(sid,semnum,SETVAL,semopts); }
注意系统调用semctl中的最后一个参数是一个联合类型的副本，而不是一个指向联合类型的指针。   　　 2.3.5 套接口
套接口（socket）编程是实现Linux系统和其他大多数操作系统中进程间通信的主要方式之一。我们熟知的WWW服务、FTP服务、TELNET服务
等都是基于套接口编程来实现的。除了在异地的计算机进程间以外，套接口同样适用于本地同一台计算机内部的进程间通信。关于套接口的经典教材同样是 Richard
Stevens编著的《Unix网络编程：联网的API和套接字》，清华大学出版社出版了该书的影印版。它同样是Linux程序员的必备书籍之一。
关于这一部分的内容，可以参照本文作者的另一篇文章《设计自己的网络蚂蚁》，那里由常用的几个套接口函数的介绍和示例程序。这一部分或许是Linux进程
间通信编程中最须关注和最吸引人的一部分，毕竟，Internet
正在我们身边以不可思议的速度发展着，如果一个程序员在设计编写他下一个程序的时候，根本没有考虑到网络，考虑到Internet，那么，可以说，他的设
计很难成功。  3 Linux的进程和Win32的进程/线程比较  　　 熟悉WIN32编程的人一定知道，WIN32的进程管理方式与Linux上有着很大区别
，在UNIX里，只有进程的概念，但在WIN32里却还有一个"线程"的概念，那么Linux和WIN32在这里究竟有着什么区别呢？
WIN32里的进程/线程是继承自OS/2的。在WIN32里，"进程"是指一个程序，而"线程"是一个"进程"里的一个执行"线索"。从核心上讲，
WIN32的多进程与Linux并无多大的区别，在WIN32里的线程才相当于Linux的进程，是一个实际正在执行的代码。但是，WIN32里同一个进
程里各个线程之间是共享数据段的。这才是与Linux的进程最大的不同。  　　 下面这段程序显示了WIN32下一个进程如何启动一个线程。  int g;
DWORD WINAPI ChildProcess( LPVOID lpParameter ){  int i;  for ( i = 1; i
<1000; i ++) {  g ++;  printf( "This is Child Thread: %d/n", g );  }
ExitThread( 0 );  };  void main()  {  int threadID;  int i;  g = 0;
CreateThread( NULL, 0, ChildProcess, NULL, 0, &threadID; );  for ( i = 1; i
<1000; i ++) {  g ++;  printf( "This is Parent Thread: %d/n", g );  }  }
在WIN32下，使用CreateThread函数创建线程，与Linux下创建进程不同，WIN32线程不是从创建处开始运行的，而是由
CreateThread指定一个函数，线程就从那个函数处开始运行。此程序同前面的UNIX程序一样，由两个线程各打印1000条信息。
threadID是子线程的线程号，另外，全局变量g是子线程与父线程共享的，这就是与Linux最大的不同之处。大家可以看出，WIN32的进程/线程
要比Linux复杂，在Linux要实现类似WIN32的线程并不难，只要fork以后，让子进程调用ThreadProc函数，并且为全局变量开设共享
数据区就行了，但在WIN32下就无法实现类似fork的功能了。所以现在WIN32下的C语言编译器所提供的库函数虽然已经能兼容大多数
Linux/UNIX的库函数，但却仍无法实现fork。
对于多任务系统，共享数据区是必要的，但也是一个容易引起混乱的问题，在WIN32下，一个程序员很容易忘记线程之间的数据是共享的这一情况，一个线程修
改过一个变量后，另一个线程却又修改了它，结果引起程序出问题。但在Linux下，由于变量本来并不共享，而由程序员来显式地指定要共享的数据，使程序变
得更清晰与安全。  至于WIN32的"进程"概念，其含义则是"应用程序"，也就是相当于UNIX下的exec了。  　　 Linux也有自己的多线程函数pth
read，它既不同于Linux的进程，也不同于WIN32下的进程，关于pthread的介绍和如何在Linux环境下编写多线程程序我们将在另一篇文章《Linu
x下的多线程编程》中讲述。

