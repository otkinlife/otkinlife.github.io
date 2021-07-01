---
title: Socket简单实现客户端与服务端交互
date: 2021/5/24 18:17:00
tags: 
    - 基础知识
categories: 网络
---

## Socket简单实现客户端与服务端交互

### 用到的系统调用方法

> - socket 的文件表描述符（fd）由五元组来唯一标识，分别是protocol, server_ip, server_port, client_ip, client_port。这5元组只要有一个不同就代表不同的fd。
> - socket 的fd的inode指向的是内存里的一个结构数据。
> - 下面提到的所有方法的介绍（包括入参和返回值）都可以使用`man`命令查看，这里不多赘述。  

#### 1. socket()

```shell
int socket(int domain, int type, int protocol);
```
- 该方法主要定义协议相关的信息，并返回一个socketfd, 此时该fd只含有protocol的信息。
  
#### 2. bind()

```shell
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
- 该方法将server_ip, server_port两部分信息写入socketfd
  
#### 3. listen()

```shell
int listen(int sockfd, int backlog);
```
- 该方法将socket()和bind()构建的fd（包含了3个元组的信息）告诉内核，使内核开始监听数据。
- 该方法不会阻塞进程，内核基于每个socket监听端口维护两个队列：已完成链接队列和未完成链接队列。
- 客户端通过connect()方法发起链接时，内核会先将该链接信息放到未完成链接队列之中，等链接完成三次握手之后再移到已完成链接队列的尾部

#### 4. accept()

```shell
int accept4(int sockfd, struct sockaddr *addr, socklen_t *addrlen, int flags);
```
- 该方法从已完成链接的队列头部取出一个链接fd
- 如果已完成链接队列为空则会阻塞，直到拿到一个链接fd

#### 5. connect()

```shell
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
- 该方法用户客户端向服务端发起链接请求

### Socket的系统链路流程图

<img src="{{site.url}}/images/blog/socket-simple-1.jpg" width="500px">

### 实现代码

#### 客户端代码

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

int main(){
    //创建套接字
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    //向服务器（特定的IP和端口）发起请求
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));  //每个字节都用0填充
    serv_addr.sin_family = AF_INET;  //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  //具体的IP地址
    serv_addr.sin_port = htons(9999);  //端口
    connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
   
    //读取服务器传回的数据
    char buffer[40];
    read(sock, buffer, sizeof(buffer)-1);
   
    printf("Message form server: %s\n", buffer);
   
    //关闭套接字
    close(sock);
    return 0;
} 
```

#### 服务端代码

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
    //进入监听状态，等待用户发起请求
    //这里的20是未完成链接队列和完成链接队列的总和
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