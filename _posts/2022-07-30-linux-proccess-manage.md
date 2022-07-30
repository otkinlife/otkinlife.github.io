---
layout: post
title: Linux：进程管理
date: 2022-07-30
tags: ["Linux","Os"]
categories: 操作系统
---


## 进程
进程是处于执行期的程序以及执行期间所涉及的资源数据（比如进程的打开文件、信号量等）。

### task_struct

linux为每一个进程维护了一个task_struct结构，该结构包含了进程运行时的状态以及涉及的资源。可以说通过task_struct这个结构可以更直观的了解一个进程。下图是task_struct的结构示意图：

<img src = "{{site.url}}/images/blog/2022-07-30-linux-process-manage/1.jpg" width = "60%">

### task_list

而包含task_struct的第一个地方是task_list，task_list以双向循环链表的形式保存在内核中。task_list用于CPU的调度，当CPU唤醒进程时就是通过该结构找到对应进程。结构如下：

<img src = "{{site.url}}/images/blog/2022-07-30-linux-process-manage/2.jpg" width = "60%">

### thread_info

task_struct由内核内存分配机制（slab分配器）分配，slab会在进程内核栈的栈顶或者栈底创建一个thread_info结构体，这个结构体就包含task_struct。这也是第二个包含task_struct的地方。该结构体的作用主要是为了便于内核快速定位到进程涉及的资源信息。（当然有的硬件体系结构可以拿出一个专门的寄存器来存放当前进程的task_struct指针，这样就不需要在内核栈中创建thread_info了），下图是thread_info在内核栈中的示意图：

<img src = "{{site.url}}/images/blog/2022-07-30-linux-process-manage/3.jpg" width = "60%">

当然除了上述重点说明的两个地方，还有其他的地方都包含task_struct，比如：task_struct结构体内部的parent和child字段等。

## 进程状态

### 状态说明

|状态|说明|
|---|---|
|TASK RUNNING（运行）|进程是可执行的：它或者正在执行，或者在运行队列中等待执行。这是进程在用户空间中执行的唯一可能的状态这种状态也可以应用到内核空间中正在执行的进程。|
|TASK INTERRUPTIBLE（可中断）|进程正在睡眠（也就是说它被阻塞），等待某些条件的达成。一旦这些条件达成，内核就会把进程状态设置为运行。处于此状态的进程也会因为接收到信号而提前被唤醒并随时准备投入运行。|
|TASK UNINTERRUPTIBLE（不可中断）|除了就算是接收到信号也不会被唤醒或谁备投入运行外，这个状态与可打断状态相同。这个状态通常在进程必须在等待时不受干扰或等待事件很快就会发生时出现。由于处于此状态的任务对信号不做响应，所以较之可中断状态，使用得较少。|
|TASK TRACED（追踪）|被其他进程跟踪的进程，例通过 ptrace 对调试程序进行跟踪。|
|TASK STOPPED（停止）|进程停止执行；进程没有投入运行也不能投入运行。通常这种状态发生在接收到SIGSTOP、SIGTSTP、SIGTTIN、SIGTTOUT等信号的时候。此外，在调试期间接收到任何信号，都会使进程进入这种状态。|

### 状态转换

<img src = "{{site.url}}/images/blog/2022-07-30-linux-process-manage/4.png" width = "60%">

## 进程创建与结束

### 进程创建

当用户调用fork()函数时，linux会将当前的进程进行copy并使用copy的资源创建一个新的子进程。子进程与父进程除了进程ID不同之外其他资源都是相同。

当要注意的是fork()函数只是创建了task_struct而不是直接运行了子进程，想要运行子进程的代码还需要执行exec()函数。

### 写时拷贝
linux为了避免不必要的浪费在实现fork()时采用了“写时拷贝”，也就是说如果资源在子进程中只读的时候是不会copy的，还是读取共享跟父进程同样的数据。只要子进程需要修改资源内容时才会进行数据拷贝。

### 进程结束

进程调用exit()来结束自己的生命。exit()方法做了几个事情：
1. 将task_struct中的flag设置为PE_EXITING。
2. 释放内存，关闭占用资源
3. 通知父进程，并寻找养父进程（养父是线程组中的其他线程或者init进程），然后将task_struct中的exit_state设置为EXIT_ZOMBIE
4. 养父进程得到通知后检索该进程的数据并释放最后的task_struct

## 进程与线程

线程是CPU调度的单位，那么linux中的线程是怎么实现的呢？
linux会在线程创建时为每个线程创建一个task_struct结构体，并在创建task_struct时指定共享的资源。那么这些共享相同资源的线程就是一个线程组，这个线程组在用户态的视角就是进程了。
所以在内核中是没有进程和线程的区分的，所谓的进程其实是线程组，CPU在调度的时候只会切换线程, 如果切换线程时恰好是切换了线程组，此时在用户态来看就切换了进程。


---
本文参考**《Linux 内核设计与实现》**第3版