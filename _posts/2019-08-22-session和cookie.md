---
layout: post
title: session 和 cookie
categories: 19th
description: cookie
keywords: cookie

---

这类问题的答案已经数不胜数，但是每次想起来都会下意识地觉得cookie和session是必须共存的，一个在客户端，一个在服务端，用来保存用户的状态信息。但是实际上，cookie和session只是两个概念，他们可以一起工作，也可以独立出来，甚至有时cookie和session在数值上是相等的。

>web开发发展至今，cookie和session的使用已经出现了一些非常成熟的方案。在如今的市场或者企业里，一般有两种存储方式：
1、存储在服务端：通过cookie存储一个session_id，然后具体的数据则是保存在session中。如果用户已经登录，则服务器会在cookie中保存一个session_id，下次再次请求的时候，会把该session_id携带上来，服务器根据session_id在session库中获取用户的session数据。就能知道该用户到底是谁，以及之前保存的一些状态信息。这种专业术语叫做server side session。
2、将session数据加密，然后存储在cookie中。这种专业术语叫做client side session。flask采用的就是这种方式，但是也可以替换成其他形式。

参考：[https://www.cnblogs.com/xxtalhr/p/9053906.html](https://www.cnblogs.com/xxtalhr/p/9053906.html)

