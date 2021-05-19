---
title: Epoll的实现原理（一）学会使用epoll
date: 2020/4/23 20:46:25
tags: 
    - 基础知识
categories:
    - 技术
    - I/O 多路复用
---

## Epoll的实现原理（一）学会使用epoll
<br/>

> 我们都知道，Epoll是解决C10K问题的利器。使用Epoll方法能够实现I/O多路复用</br>
> 本文结合实践剖析Epoll底层是如何实现的


### 1. 学习Epoll的关键
Epoll是Linux lib库里的方法，其中关键的三个方法:
- epoll_create()
- epoll_ctl()
- epoll_wait()
  
可以说掌握了这三个方法的原理就已经掌握90%的Epoll了

### 2. 如何使用Epoll方法

我们先通过实践的方式来了解一下Epoll的用法，然后才能更好了解其中的原理

#### 简单介绍一下Epoll的三个方法（以下介绍皆来自与man命令）

##### epoll_create
```c
//创建一个epoll句柄
 int epoll_create(int size);
```
1. 入参size： 一个大于0的整数，从Linux 2.6.8之后该参数并没有实际作用
2. 返回成功：一个大于0的值，表示epoll实例；该实例将会作为句柄被epoll_ctl()和epoll_wait()方法使用
3. 返回失败：-1

##### epoll_ctl
```c
//基于创建的epoll实例增加监听事件
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```
1. 入参epfd：epoll_create创建的句柄
2. 入参op：创建的事件操作方式，可选项如下
   - EPOLL_CTL_ADD 增加事件
   - EPOLL_CTL_MOD 修改事件
   - EPOLL_CTL_DEL 删除事件
3. 入参fd： 注册的事件fd，即需要监听的fd
4. 入参epoll_event：关注的事件类型，即触发什么条件才会通知进程。可选项如下
   - EPOLLIN：表示对应的文件描述字可以读
   - EPOLLOUT：表示对应的文件描述字可以写
   - EPOLLRDHUP：表示套接字的一端已经关闭，或者半关闭
   - EPOLLHUP：表示对应的文件描述字被挂起
   - EPOLLET：设置为 edge-triggered，默认为 level-triggered
5. 返回成功：0
6. 返回失败：-1

##### epoll_wait
```c
//阻塞等待唤醒
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout); 
```
1. 入参epfd：epoll_create创建的句柄
2. 入参epoll_event：返回需要被唤醒的事件，相当于传一个空的对象指针进去，被唤醒时会被写入唤醒的事件数据
3. 入参maxevents：一个大于 0 的整数，表示 epoll_wait 可以返回的最大事件值
4. 入参timeout：阻塞调用的超时值，如果这个值设置为 -1，表示不超时；如果设置为 0 则立即返回，即使没有任何 I/O 事件发生
5. 返回成功：一个大于0的数，表示事件的个数
6. 返回失败：-1

#### 如何使用epoll进行事件监听