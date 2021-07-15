---
layout: post
title: Linux：性能查看工具
date: 2021-07-15
tags: ["Linux","Os"]
categories: 操作系统
---

### Linux 性能查看工具

#### 什么是平均负载（load average）
我们在执行top/uptime时有一行load average的状态后面有3个数字，分别代表系统在1min、5min、15min的平均负载。那什么是平均负载呢，我们使用`man uptime`可以看到官方的解释如下：
> System  load  averages  is  the  average number of processes that are either in a runnable or uninterruptable state.  A process in a runnable state is either using the CPU or waiting to use the CPU.  A process in uninterruptable state is waiting for some I/O access, eg waiting for disk.  The averages are taken over the three time intervals.  Load averages are not normalized for the number of CPUs in a system, so a load average of 1 means a single CPU system is loaded all the time while on a 4 CPU system it means  it was idle 75% of the time.

大概意思是load average 是系统处于运行和不可中断进程的平均数，所以如何判断系统是否超载就看这3个数值/CPU数即可。活跃的进程数量如果刚刚等于CPU数量是最佳的情况。

注意我们刚刚说的是系统处于运行和不可中断的进程，这就说明如果负载过高有可能是使用CPU的进程数量过高（这也包括正在发生CPU调度的进程）或者进行I/O的进程数过高

#### 如何找到负载过高的瓶颈
如果我们发现负载过高，一般会使用sysstat工具集来定位原因。sysstat是一个工具集合包含了用于采集linux各种运行指标的工具，安装也比较简单：

```shell
# centos
yum install -y sysstat
# ubuntu
apt install -y sysstat
```
下面就是几个常用工具的介绍：

#### 1. mpstat
> mpstat命令写入每个可用处理器的标准输出活动。还报告了所有处理器的全局平均活动。

```shell
mpstat [ -A ] [ -u ] [ -V ] [ -I { SUM | CPU | SCPU | ALL } ] [ -P { cpu [,...] | ON | ALL } ] [ interval [ count ] ]
```
信息详解：

|name|note|
|:-:|:--|
|CPU|处理器编号。关键字all表示统计数据是作为所有处理器之间的平均值计算的。|
|%usr|显示在用户级别（应用程序）执行时发生的CPU利用率百分比。|
|%nice|显示在具有更高优先级的用户级别执行时发生的CPU利用率百分比。|
|%sys|显示在系统级（内核）执行时发生的CPU利用率百分比。请注意，这不包括维护硬件和软件中断所花费的时间。|
|%iowait|显示系统有未完成的磁盘I/O请求期间CPU或CPU空闲的时间百分比。|
|%irq|显示CPU或CPU处理硬件中断所花费的时间百分比。|
|%soft|显示CPU或CPU为软件中断服务所花费的时间百分比。|
|%steal|显示虚拟机监控程序为另一个虚拟处理器提供服务时，虚拟CPU或CPU非自愿等待所花费的时间百分比。|
|%guest|显示CPU或CPU运行虚拟处理器所花费的时间百分比。|
|%gnice|显示CPU或CPU运行niced guest所花费的时间百分比。|
|%idle|显示CPU或CPU空闲且系统没有未完成的磁盘I/O请求的时间百分比。|

使用举例：

```shell
# 以两秒钟的间隔显示所有处理器中的五个全局统计信息报告。
mpstat 2 5
# 每隔两秒显示所有处理器的五个统计报告。
mpstat -P ALL 2 5
```

#### 2. pidstat
>pidstat命令用于监视当前由Linux内核管理的单个任务。对于使用选项-p选择的每个任务，或者对于由Linux内核管理的每个任务（如果选择了选项-p ALL），它都会写入标准输出活动
用过的。不选择任何任务相当于指定-p ALL，但只有活动任务（具有非零统计值的任务）才会显示在报告中。

```shell
pidstat [ -d ] [ -h ] [ -I ] [ -l ] [ -r ] [ -s ] [ -t ] [ -U [ username ] ] [ -u ] [ -V ] [ -w ] [ -C comm ] [ -p { pid [,...] | SELF | ALL } ] [ -T { TASK | CHILD | ALL } ] [ interval [ count ] ]
```
信息详解：

|name|note|
|:-:|:--|
|UID|正在监视的任务的实际用户标识号。|
|USER|拥有被监视任务的真实用户的名称。|
|PID|正在监视的任务的标识号。|
|%usr|任务在用户级（应用程序）执行时使用的CPU百分比，有或没有良好的优先级。请注意，此字段不包括运行虚拟处理器所花费的时间。|
|%system|任务在系统级（内核）执行时使用的CPU百分比。|
|%guest|任务在虚拟机（运行虚拟处理器）中花费的CPU百分比。|
|%CPU|任务使用的CPU时间的总百分比。在SMP环境中，如果在命令行中输入了选项-I，则任务的CPU使用率将除以CPU的总数。|
|CPU|任务附加到的处理器号。|
|Command|任务的命令名。|

使用举例：

```shell
# 每隔两秒显示系统中每个活动任务的五次统计报告。
pidstat 2 5
# 每隔两秒显示五个关于PID 1643的page faults和内存统计信息的报告。
pidstat -r -p 1643 2 5
# 显示命令名包含字符串“fox”或“bird”的所有进程的全局page faults和内存统计信息。
pidstat -C "fox|bird" -r -p ALL
# 以两秒钟的间隔显示系统中所有任务的子进程的五个page faults统计报告。只显示统计值非零的子进程。
pidstat -T CHILD -r 2 5
```

#### 3. iostat 
>iostat命令用于通过观察设备相对于其平均传输速率的激活时间来监视系统输入/输出设备负载。iostat命令生成可用于将系统配置更改为的报告
更好地平衡物理磁盘之间的输入/输出负载。

```shell
iostat [ -c ] [ -d ] [ -h ] [ -k | -m ] [ -N ] [ -t ] [ -V ] [ -x ] [ -y ] [ -z ] [ -j { ID | LABEL | PATH | UUID | ... } ] [ [ -T ] -g group_name ] [ -p [ device [,...] | ALL ] ] [ device [...] | ALL ] [ interval [ count ] ]
```

信息详解：

|name|note|
|:-:|:--|
|%user|显示在用户级别（应用程序）执行时发生的CPU利用率百分比。|
|%nice|显示在具有良好优先级的用户级别执行时发生的CPU利用率百分比。|
|%system|显示在系统级（内核）执行时发生的CPU利用率百分比。|
|%iowait|显示系统有未完成的磁盘I/O请求期间CPU或CPU空闲的时间百分比。|
|%steal|显示虚拟机监控程序为另一个虚拟处理器提供服务时，虚拟CPU或CPU非自愿等待所花费的时间百分比。|
|%idle|显示CPU或CPU空闲且系统没有未完成的磁盘I/O请求的时间百分比。|

使用举例：

```shell
# 为所有CPU和设备显示一个自启动以来的历史记录报告。
iostat
# 每隔两秒显示一个连续的设备报告。
iostat -d 2
# 以两秒的间隔显示设备sda和sdb的六个扩展统计报告。
iostat -x sda sdb 2 6
# 以两秒钟的间隔显示设备sda及其所有分区（sda1等）的六个报告
iostat -p sda 2 6
```
