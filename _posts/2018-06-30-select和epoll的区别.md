---
layout: post
title: select 和epoll 的区别
categories: 18th
description: select, epoll
keywords: select, epoll , 区别

---

socket编程并发处理的问题中，select和epoll函数的区别一直是面试中的重点。关于他俩的区别很多博客中已经说明了，我在这里总结一下。参考[select和epoll 原理概述&优缺点比较](https://blog.csdn.net/jiange_zh/article/details/50811553)以及[select、poll、epoll之间的区别总结[整理]](https://blog.csdn.net/qq546770908/article/details/53082870)

##select的过程

调用select函数时到底发生了什么，即如何实现同时监听多个socket的。假设我们需要监听的读套接字read[]，它作为参数传递进了select函数。

    select(fd_set read[],fd_set [],fd_set [],timeout)

1. 从用户空间拷贝fd_set到内核空间，也即从当前程序拷贝fd_set数组进内核，fd_set是什么可以参考百度百科，简单的说，是可以对socket进行操作的long型数组。
2. 对所有的fd进行一次poll操作，即把当前进程挂载到fd上。
3. poll操作过程中select会唤醒所有的队列中节点，进行遍历，得到它们的掩码（不同的掩码表示不同的就绪状态）。
4. 如果所有设备返回的掩码都没有显示任何的事件触发，就去掉回调函数的函数指针，进入有限时的睡眠状态，再恢复和不断做poll，再作有限时的睡眠，直到其中一个设备有事件触发为止。
5. 只要有事件触发，系统调用返回，将fd_set从内核空间拷贝到用户空间，回到用户态，用户就可以对相关的fd作进一步的读或者写操作了。

## epoll过程

这里我就直接引用文章[select和epoll 原理概述&优缺点比较]((https://blog.csdn.net/jiange_zh/article/details/50811553)中的话。

>调用epoll_create时，做了以下事情：
>1. 内核帮我们在epoll文件系统里建了个file结点；
> 2. 在内核cache里建了个红黑树用于存储以后epoll_ctl传来的socket；
> 3. 建立一个list链表，用于存储准备就绪的事件。

>调用epoll_ctl时，做了以下事情：
>1. 把socket放到epoll文件系统里file对象对应的红黑树上；
>2. 给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪list链表里。

>调用epoll_wait时，做了以下事情：
观察list链表里有没有数据。有数据就返回，没有数据就sleep，等到timeout时间到后即使链表没数据也返回。而且，通常情况下即使我们要监控百万计的句柄，大多一次也只返回很少量的准备就绪句柄而已，所以，epoll_wait仅需要从内核态copy少量的句柄到用户态而已。

总结一下，就是epoll不需要通过遍历的方式，而是在内核中建立了file节点，并且通过注册响应事件的方式，当有响应事件发生时采取相应的措施，并把准备就绪的事件放入链表中，从而epoll只关心链表中是否有数据即可。

##对比

很明显，select的效率低于epoll，因为它需要大量拷贝fd_set，并且需要不断遍历监听列表，而epoll这种基于响应事件的方式明显会更具优势。