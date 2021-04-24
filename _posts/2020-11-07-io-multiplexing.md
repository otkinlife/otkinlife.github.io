---
layout: post
title: I/O多路复用：EPOLL
date: 2020-11-07
tags: ["I/O多路复用","Linux"]
---

<!-- wp:heading {"level":4} -->

#### EPOLL是什么？

<!-- /wp:heading -->

<!-- wp:paragraph -->

I/O多路复用主要用于解决高并发问题，最早的时候使用的是select，select使用了一个fd数组，每次调用都会循环遍历该数组，但问题是每次主动轮询大量的消耗资源，并且linux本身对fd有限制，后来出现了poll方法，但是poll方法仅仅解决了文件描述符限制的问题，不断主动轮询带来的资源消耗依然存在。为了解决这个问题才有了我们今天要总结的epoll方法。epoll方法采用回调的方式，等待I/O数据就位后会主动唤醒进程。

<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->

#### 引子

<!-- /wp:heading -->

<!-- wp:paragraph -->

想要了解epoll是怎么工作的首先要明确一件事：在linux内核里每一个资源都对应一个等待队列（wait_queue_head_t），当进程需要等待该资源的数据时，可以主动调用内核的poll方法，将该进程挂到等待队列中。等到数据就绪后会通过中断的方式告诉CPU，唤醒该进程继续工作。

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

内核的poll方法（f_op->poll）默认做的事情是把睡在等待队列的进程唤醒，当然该方法支持我们自定义。

<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->

#### 运行原理

<!-- /wp:heading -->

<!-- wp:paragraph -->

epoll核心的三个方法epoll_create()，epoll_ctl()，epoll_wait()。

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

epoll_create

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

该方法创建了一个eventpoll的对象，该对象是以一个文件的形式存在，主要的数据结构，一个是用于维护监听的文件描述符状态的红黑树rbr，一个是用于唤醒进程的就绪队列，还有一个是阻塞与epoll_wait()的等待队列

<!-- /wp:paragraph -->

<!-- wp:image {"id":346,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large">![](image-4.png)<figcaption>eventpoll</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->

epoll_ctl

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

该方法先行找到eventpoll对象，在其内部的红黑树中确认是否已经包含该节点，如果没有则需要新增。主要做了以下几件事

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1.  在红黑树中新增一个epitem节点
2.  调用内核的poll方法将进程挂到该资源的等待队列中去，并设置自定义的回调方法ep_poll_callback。（这里调用poll方法的时候可能资源已经就绪，这种情况会在返回值里表现出来，epoll_ctl会对其各种情况做处理，因为这属于特殊情况处理所以不多赘述）。
3.  ep_poll_callback回调的核心逻辑是，当资源就绪后将该资源对应的epitem加入就绪队列中
<!-- /wp:list -->

<!-- wp:paragraph -->

所以在epoll_ctl的时候epoll就已经开始监听资源了

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

epoll_wait

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

该方法将监控rdlist就绪队列，如果不为空的时候会将rdlist的句柄通过ep_send_events方法复制到用户空间。ep_send_events方法会调用默认的内核poll方法唤醒进程。

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

补充：关于LT和ET的不同也是在这一步体现，如果是LT会把该epitem再次加入到rdlist里

<!-- /wp:paragraph -->