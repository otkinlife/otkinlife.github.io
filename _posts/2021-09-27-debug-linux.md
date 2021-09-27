---
title: 如何Debug Linux内核
date: 2021/9/27 14:34:00
tags: 
    - 操作系统
categories:
    - 技术
    - 操作系统
---

## 如何Debug Linux内核

### 引子
最近在看linux内核相关的东西，但是对着资料看源码总是觉得差一点东西。无疑没有Debug看源码对于我们的理解有很大的挑战，所以想一边debug一边学习，查了很多资料，磕磕绊绊终于成功了。下面以arm_64架构为例把我踩的坑和经验总结出来。
> 目前已将调试环境打包成了docker镜像，以后想要调试内核可以直接拉取镜像
> 
> 镜像的地址是：
> 
>  1. arm_64：https://hub.docker.com/repository/docker/appledate/linux-arm-qemu
>  2. x86_64：https://hub.docker.com/repository/docker/appledate/linux-x86_64-qemu 

### Debug 内核的基本逻辑
首先我们 要搞清的几件事：

1. 内核在启动之后本质上也是一个进程， 我们想要debug内核其实就是debug一个进程
2. 我们要在一个系统启动的时候设置好输出调试信息，所以我们需要一个虚拟机（我们这里用qemu作为虚拟机）。使用虚拟机启动内核并开启调试，再利用宿主机去调试。
3. 一个系统的运行需要编译好的内核镜像和一个文件系统，当然还得安装一个虚拟机

### 动手搞起来

#### 下载并编译linux内核
我这里用的linux-5.13.8版本

```bash
#进入工作目录
cd ~
#下载linux源码
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.13.8.tar.gz
# 解压linux源码
tar -zxf linux-5.13.8.tar.gz
# 安装依赖库
apt install build-essential flex bison libssl-dev libelf-dev libncurses-dev bc -y
# 进入源码目录
cd linux-5.13.8
# 配置linux的编译配置，linux内核编译的配置方式有很多，大家有兴趣可以网上查一下
make defconfig
# 编译内核
make -j4

```
2000 years later...

编译完成之后我们可以看到
目录下~/linux-5.13.8/arch/arm64/boot/应该有Image相关的文件

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/1.png" width = "60%">

此时编译内核就完成了
 
#### 构建根文件系统
我们使用busybox工具构建根文件系统，这里选择busybox是因为busybox本身就是一个工具集合，内置了大量常用命令，虚拟机启动之后会很方便

```bash
#回到工作目录
cd ~
#下载busybox源码
wget https://busybox.net/downloads/busybox-1.34.0.tar.bz2
#解压busybox源码
tar -xf busybox-1.34.0.tar.bz2
# 进入busybox目录
cd busybox-1.34.0
# 配置编译配置: Setting->Build options->Build static binary (no shared libs)
# 配置完成之后编译安装
make && make insatll
```
安装完成之后出现：

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/2.png" width = "60%">

此时安装好的文件应该在~/busybox-1.34.0/_install下

接下来开始构建文件镜像：

```bash
# 进入工作目录
cd ~

# 创建镜像文件夹，该目录里面的所有文件都将被打包成镜像，作为虚拟机启动的根目录
mkdir myrootfs && cd myrootfs 

# 将刚刚安装好的busybox下的_install复制到rootfs下
cp ../busybox-1.34.0/_install/* ./ -rf

#创建linux系统运行时必要的挂载点
mkdir proc sys dev etc etc/init.d
#开始构建rcS脚本，rcS是linux内核刚启动时需要执行的脚本
echo '#!/bin/sh' > etc/init.d/rcS
echo "mount -t proc none /proc" >> etc/init.d/rcS
echo "mount -t sysfs none /sys" >> etc/init.d/rcS
echo "/sbin/mdev -s" >> etc/init.d/rcS
chmod +x etc/init.d/rcS

#将myrootfs打包成镜像
apt install cpio -y
find . | cpio -o --format=newc > ../rootfs.img
```
 这一步过后在工作目录（也就是myrootfs的上级目录）应该会有一个rootfs.img文件
 
 #### 使用qemu启动linux内核
 
 ```bash
 #进入工作目录
 cd ~
 # 安装qemu
 apt install -y qemu-system
 # 使用qemu启动linux内核
 qemu-system-aarch64 \
 -machine virt,virtualization=true,gic-version=3 \
 -nographic \
 -cpu cortex-a57 \
 -kernel /root/linux-5.13.8/arch/arm64/boot/Image \
 -initrd /root/rootfs.img \
 --append "console=ttyAMA0 rdinit=/linuxrc nokaslr"
 ```
 此时进入了qemu的虚拟机内核，看到如下界面说明已经进入到qemu启动的虚拟机里了
 
<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/3.png" width = "60%">

如果想退出虚拟机可以执行命令

`poweroff`

#### 开始调试

我们只需要在启动qemu虚拟机时追加 -S -s参数即可开启Debug模式，此时qemu会在启动虚拟机的同时开启一个gdbserver，端口号是1234

```bash
#安装gdb
apt install gdb -y
#以调试模式启动qemu之后会立马进入阻塞等待gdb连接
 qemu-system-aarch64 \
 -machine virt,virtualization=true,gic-version=3 \
 -nographic \
 -cpu cortex-a57 \
 -kernel /root/linux-5.13.8/arch/arm64/boot/Image \
 -initrd /root/rootfs.img \
 --append "console=ttyAMA0 rdinit=/linuxrc nokaslr" \
 -S -s
```
此时我们可以使用gdb进行连接调试了，要注意的是gdb启动的时候需要跟一个符号文件，该文件是在第一步编译内核时产生的/root/linux-5.13.8/vmlinux

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/4.png" width = "80%">

```bash
#启动gdb
gdb /root/linux-5.13.8/vmlinux
#此时进入了gdb的交互界面
#连接gdbserver
target remote :1234 
# 在初始化内核处打断点
b start_kernel
# 执行到断点处
c
```
<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/5.png" width = "80%">

此时可以（愉快的？）Debug内核了

#### 使用Clion远程调试
命令行调试效率太低，加上ide才能如虎添翼
我这里以clion为例，当然其他ide也大同小异

我们依然以调试模式启动qemu

```bash
#以调试模式启动qemu之后会立马进入阻塞等待gdb连接
 qemu-system-aarch64 \
 -machine virt,virtualization=true,gic-version=3 \
 -nographic \
 -cpu cortex-a57 \
 -kernel /root/linux-5.13.8/arch/arm64/boot/Image \
 -initrd /root/rootfs.img \
 --append "console=ttyAMA0 rdinit=/linuxrc nokaslr" \
 -S -s

```
在本地下载好linux内核源码（当然这里也可以直接从你搭建环境的服务器上拉取，看个人喜好），并用clion打开。
接下来开始配置clion
点开右上角的debug配置界面，然后点击+号新增一个GDB Remote Debug

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/6.png" width = "80%">

按照实际情况进行配置
1. GDB：一般用clion默认的即可，也可以使用自己安装的
2. targe remote args：有调试环境的服务地址，一般格式为 tcp:xxxxx:1234
3. Symol file：这个很重要，要把第一步中编译生成的vmlinux符合文件同步到clion所在的机器上然后配置

配置完成后点击debug图标即可开始debug了

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/7.png" width = "80%">

#### 如何Debug具体的系统调用

最后说一下如何去debug某些具体的系统调用，比如我希望debug一下socket_create这个方法

首先我们可以自己写一个demo，下面是我写的调用socket的小例子:

server.cpp

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int serv_sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    //将套接字和IP、端口绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));  //每个字节都用0填充
    serv_addr.sin_family = AF_INET;  //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  //具体的IP地址
    serv_addr.sin_port = htons(9999);  //端口
    bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    printf("start listening ... \n");
    //进入监听状态，等待用户发起请求
    listen(serv_sock, 20);

    //接收客户端请求
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size = sizeof(clnt_addr);
    int clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_addr, &clnt_addr_size);
    //向客户端发送数据
    char str[] = "hello world";
    write(clnt_sock, str, sizeof(str));
   
    //关闭套接字
    close(clnt_sock);
    close(serv_sock);
    return 0;
   
}
```
我们进入搭建好的linux调试环境

```bash
#进入文件镜像目录
cd /root/my-rootfs
#新建一个工程目录，这个就是为了找个地方写代码
mkdir developer && cd developer
#新建server.cpp，并将上面的demo代码贴入
# 编译server.cpp
gcc -o server -static server.cpp
#回到文件镜像目录
cd /root/my-rootfs
#重新打包镜像
find . | cpio -o --format=newc > ../rootfs.img
```
我们重新打包了镜像，此时只要我们以调试模式启动qemu就行了

```bash
#以调试模式启动qemu之后会立马进入阻塞等待gdb连接
 qemu-system-aarch64 \
 -machine virt,virtualization=true,gic-version=3 \
 -nographic \
 -cpu cortex-a57 \
 -kernel /root/linux-5.13.8/arch/arm64/boot/Image \
 -initrd /root/rootfs.img \
 --append "console=ttyAMA0 rdinit=/linuxrc nokaslr" \
 -S -s

```
然后我们使用gdb连接上gdbserver并直接进入console（只要我们连接上gdbserver的时候没有打断点即可）

进入console之后，我们开始打断点

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/8.png" width = "80%">

下面我们在qemu启动的终端操作
```bash
#进入 我们之前建的developer目录
cd /developer
# 启动编译好的server
./server
```

<img src = "{{site.url}}/images/blog/2021-09-27-debug-linux/9.png" width = "80%">

到此我们就可以调试自定义的程序了

参考文档：[https://wenfh2020.com/2021/05/19/gdb-kernel-networking](https://wenfh2020.com/2021/05/19/gdb-kernel-networking)