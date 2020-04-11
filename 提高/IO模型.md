### UNIX的5种IO模型

__UNIX IO模型矩阵__

![unix IO模型矩阵图](E:\DarkhorseRoad\github\priv\network\images\QQ截图20190928114740.png)

1. 阻塞IO
   + 应用程序调用一个IO函数，导致应用程序阻塞，如果数据已经准备好，从内核拷贝到用户空间，否则一直等待下去。
   + 非阻塞IO
   + IO复用（select和poll）
   + 信号驱动IO
   + 异步IO（Posix.1的aio_函数）
2. IO操作的一个输入操作分为两步：
   + 用户进程切换道内核进程，等待网络数据传输完成，放置在内核的缓冲区
   + 将数据从内核缓冲区拷贝到应用程序缓冲区，内核进程切换回用户进程

#### 阻塞I/O（block IO)

+ 应用程序调用IO函数（recvfrom），导致应用程序阻塞，等待数据传输完成，从内核拷贝到用户控件，否则一直等待下去。__linux中，默认情况下所有的socket都是blocking__

+ blocking IO在IO执行的两个阶段都是被block

+ Ex：小明拿着水杯去水龙头接水，水龙头不出水，小明就一直拿着水杯等着，直到水龙头出水，接满水然后离开，整个过程小明不能干其他事。

+ 阻塞IO的读操作流程：

  ![阻塞IO读操作流程](E:\DarkhorseRoad\github\priv\network\images\QQ截图20190928120131.png)



#### 非阻塞IO(nonblocking IO)

+ 套接口设置为非阻塞，就是告诉内核，当所有的IO操作无法完成时，不要将进程睡眠，而是返回一个错误。IO操作函数（recvfrom）不断的进行测试数据是否已经准备好了，没有准备好就继续测试直到数据准备好了，在这个过程中，会大量的占用CPU的时间。

+ 流程

  + 用户进程不断的切换为内核进程，查看数据是否已经传输完成，（__该过程称作轮询__（poiling））,会大量占用CPU的时间
  + 数据准备好了，内核进程把数据从内核缓冲区拷贝到应用缓冲区，该过程需要时间，所有说非阻塞IO是同步的<同步非阻塞>

  ![非阻塞IO的读操作流程](E:\DarkhorseRoad\github\priv\network\images\QQ截图20190928120856.png)

  

#### IO多路复用模型（IO multiplexing）

+ 阻塞IO模式下，一个线程只能处理一个流的IO事件，如果想要处理多个流，要么使用多进程，要么使用多线程，很不幸这两种方法效率都不是很高，于是再考虑非阻塞忙轮询的IO方式，就可以同时处理多个流的IO事件了，只需要把所有的流从头到尾测试一变，再从头开始，这样就可以处理多个流了，但是如果所有的流都没有数据，只会白白浪费CPU。

+ 引进代理（select,poll),可以同时观察许多流的IO事件，空闲的时候，会把线程阻塞掉，当有一个或者多个流有IO事件时，就会从阻塞状态醒来，于是程序就可以轮询一遍所有的流。

+ 流程：

  ![io多路复用模式的流程](E:\DarkhorseRoad\github\priv\network\images\IO多路复用模式流程.png)

+ 多路复用模式和阻塞IO模式没有太大区别，区别就是多路复用模式需要进行两次system_call(recefrom和select)，阻塞IO只调用了一次（recefrom）。
+ select也会阻塞进程，如果处理连接数不高的话，使用select/epoll不一定比使用多进程+阻塞IO效率高，select的优势在于可以处理多个connection
+ 在IO多路复用模式中，对于每一个socket，都是设置成non-blocking,这里的阻塞是被select函数阻塞的，不是被Socket io阻塞的。



#### **信号驱动I/O模型**

+ 我们也可以用信号，让内核在描述符就绪时发送SIGIO信号通知我们。通过sigaction系统调用安装一个信号处理函数。该系统调用将立即返回，我们的进程继续工作，也就是说它没有被阻塞。当数据报准备好读取时，内核就为该进程产生一个SIGIO信号。我们随后既可以在信号处理函数中调用recvfrom读取数据报，并通知主循环数据已经准备好待处理。

+ 优点：等待数据报到达期间进程不被阻塞。主循环可以继续执行，只要等待来自信号处理函数的通知：既可以是数据已准备好被处理，也可以是数据报已准备好被读取。

+ 流程：

  ![信号驱动模型IO流程](E:\DarkhorseRoad\github\priv\network\images\信号驱动IO模型流程.png)

#### 异步IO模型（asynchronous IO）

+ 告知内核启动某个操作，并让内核在整个操作(包括将内核复制到我们自己的缓冲区)完成后通知我们。

+ 调用aio_read（Posix异步I/O函数以aio_或lio_开头）函数，给内核传递描述字、缓冲区指针、缓冲区大小（与read相同的3个参数）、文件偏移以及通知的方式，然后系统立即返回。我们的进程不阻塞于等待I/0操作的完成。当内核将数据拷贝到缓冲区后，再通知应用程序。

+ 用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

+ 流程：

  ![异步IO流程](E:\DarkhorseRoad\github\priv\network\images\异步IO流程.png)





#### 比较

1. blocking 和non-blocking的区别
   + 调用blocking IO会一直block对应的应用进程直到操作完成，而non-blocking 在内核还未准备好数据的情况下回立即返回。
2. synchronous和asynchronous IO的区别
   + 两者的区别在于同步IO做IOoperation的时候会将process阻塞，所以blockingIO，non-blockingIO，IO multiplexing，信号驱动模式IO都属于同步IO。
   + 这里的IO操作指的是真实的IO操作，就是recefrom这个system_call。多路复用IO模型在调用recefrom时，当内核没有准备好数据的时候，不会阻塞进程，但是当数据准备好后，recefrom会将数据从内核缓冲区拷贝到应用缓冲区，这个过程是阻塞的。
   + 异步IO当进程发起IO操作之后，就直接返回不再理睬了，知道内核发送一个信号，告诉进程，IO完成，整个过程中进程都没有被阻塞。

