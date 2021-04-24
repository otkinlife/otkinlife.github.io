---
layout: post
title: Redis应用：队列
date: 2020-07-04
tags: ["Redis","Redis"]
categories: Redis

---

<!-- wp:paragraph -->

由于Redis中List结构基于链表实现，其插入删除非常高效（O(1)）。并且Redis本身就支持List的pop和push操作，所以使用List去实现队列或者栈这两种结构都非常合适。

<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->

#### 队列：

<!-- /wp:heading -->

<!-- wp:heading {"level":5} -->

##### 使用场景

<!-- /wp:heading -->

<!-- wp:paragraph -->

首先要明确一件事，什么情况下要使用Redis去实现一个队列而不是使用市场上已经很成熟的第三方服务，比如Kafka或者RabbitMQ。我们知道第三方服务有很多优势，像消息的可靠性、完整性、安全性，又能支持多场景的定制化需求。这样的优点同时也大大增加了维护成本，如果我们现在的需求对消息的要求并不严格，可以允许丢失的情况下，Redis无疑是很好的选择。

<!-- /wp:paragraph -->

<!-- wp:heading {"level":5} -->

##### 实现

<!-- /wp:heading -->

<!-- wp:code -->

    # 入队列
    rpush testQueue msg1
    # 出队列
    lpop testQueue
    # 查看队列长度
    llen testQueue

<!-- /wp:code -->

<!-- wp:paragraph -->

但是正常的情况下，队列的消费者是要长期在后台运行的，通过不停的pop获取消息。这个时候如果列被消费完了我们还在不断的pop的话，就会产生不必要的资源浪费。所以我们改进一下上述的写法，使用阻塞读。

<!-- /wp:paragraph -->

<!-- wp:code -->

    # 阻塞读出队列
    blpop testQueue 1000

<!-- /wp:code -->

<!-- wp:paragraph -->

这样一个简单的队列服务就完成了。当然如果想要在线上应用还要加上日志监控，异常处理机制等等，这些都可以交给服务端，使用逻辑去完善。

<!-- /wp:paragraph -->