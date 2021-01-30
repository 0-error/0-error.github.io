---
layout: post
title: select 函数socket 编程
categories: 18th
description: select 函数
keywords: select 函数

---

套接字编程中的select函数是一个比较重要的概念，是epoll函数的早期版本，是实现I/O复用的关键方法。在对这个方法的了解过程中，走了一些弯路，由于select一直与并发，异步等字眼联系在一起，总觉得select是不阻塞的，事实上它本身是阻塞的。当我苦苦在网上搜索答案却相互矛盾的时候，我知道是时候冷静下来喝一口水，思考自己是否还适合干这一行。不对，是是时候找一本权威的书籍翻阅一下了。
##阻塞I/O模型
首先需要了解的是socket阻塞的概念，我们知道socket的读写操作并不是直接向网络中读写，而是向本地的网络缓冲区中读写，每一个socket会被分配一个读和一个写缓冲区，如果当我们希望读取数据，但是缓冲区中还没有数据的时候，那么读方法（recv）会被阻塞知道有足够的数据可以接受，那么这个过程中进程会被阻塞，你的程序不能干别的，只能等。在《UNIX网络编程》一书中，用这样的图来表示这个过程。
![image.png](https://upload-images.jianshu.io/upload_images/2360187-4c758e4a5b9b4419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##I/O复用模型
第二个需要了解的概念是I/O复用，因为select，包括poll和epoll都是实现这种模型的机制，我不明白为什么要叫I/O复用，但是它的好处是进程阻塞与select函数，而不是阻塞于真正的I/O系统调用，那么这个过程同样书中描述为
![image.png](https://upload-images.jianshu.io/upload_images/2360187-50eb6b0a0ddd0368.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看两张图，好像复用并没有什么优越性，因为它同样要阻塞等待直到在监听的socket列表中有socket读或写操作返回。时间上好像并没有什么差别。

##select的优越性
select的优越性主要表现在它可以实现一个线程监听多个socket句柄，而阻塞模型则需要多线程来达到这个目的，所以在并发的大量请求的情况下，这样是能节省很多开销的。还有在设置好timeout后，超过这个时间select将不再阻塞，会返回错误，这样可以接着执行别的操作，一定程度上能达到异步的目的。所以你看，这样就和并发和异步联系上了。

##结尾
 还有很多问题比方说select监听多套接字机制，轮询的方法，就不探讨了，有兴趣可以自己看下。