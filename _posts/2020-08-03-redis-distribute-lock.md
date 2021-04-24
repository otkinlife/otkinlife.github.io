---
layout: post
title: Redis应用：分布式锁
date: 2020-08-03
tags: ["Redis","Redis"]
categories: Redis

---

<!-- wp:heading {"level":4} -->

#### 分布式锁是什么

<!-- /wp:heading -->

<!-- wp:paragraph -->

在过去的年代，因为用户量少，我们部署一个页面只需要一台服务器。但随着用户量的急剧上升，单机的性能升级已无法满足需求。于是人们提出使用多台机器共同完成一个服务的计算，这就是分布式的由来。在单机上运行一个程序我们可能不需要考虑一致性和资源竞争的问题，但是多台机器上数据如何协调，进程之间如何互相独立的同时又能协作完成工作呢。在一个操作系统中进程间竞争资源是依靠锁的，我们借鉴这个思想，多机部署的服务也同样可以使用锁机制来协调资源，也就是分布式锁

<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->

#### 使用Redis实现分布式锁

<!-- /wp:heading -->

<!-- wp:heading {"level":5} -->

##### 核心思想：

<!-- /wp:heading -->

<!-- wp:paragraph -->

使用Redis实现分布式锁的核心思想是使用setnx命令在内存中写入一个key（这个key一般指锁的名称），当其他服务或者进程想要访问某些资源的时候，会尝试调用setnx命令，如果写入成功则说明锁没有被占用，并自己占住该锁，否则就进入阻塞状态等待锁释放。

<!-- /wp:paragraph -->

<!-- wp:code -->

    # 加锁
    setnx lock_test1 true //setnx命令是如果不存在再写入
    # 释放锁
    del lock_test1

<!-- /wp:code -->

<!-- wp:heading {"level":5} -->

##### 问题：服务宕机导致所无法释放

<!-- /wp:heading -->

<!-- wp:paragraph -->

简单实现了Redis版的分布式锁后会发现：如果一个服务加了一个锁之后挂掉了，会导致这个锁永远不能释放，为了解决这个问题我们需要在加锁的同时设定一个过期时间，如果长期没有释放的锁直接过期删除。

<!-- /wp:paragraph -->

<!-- wp:code -->

    # 进阶版加锁
    set lock_test1 true ex 10 nx //加锁并设置超时时间
    # 释放锁
    del lock_test1

<!-- /wp:code -->