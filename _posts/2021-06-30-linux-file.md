---
layout: post
title: Linux：打开文件
date: 2021-06-30
tags: ["Linux","Os"]
categories: Linux

---
## Linux打开文件
### 引子
#### linux的磁盘是如何存储文件的
##### 存储结构inode和block
Linux文件系统将磁盘划分为块（block），一个文件被存放在多个块中。为了能找到存放某个文件的所有块，Linux为每个文件维护了一个inode结构，该结构如下：

```c
struct ext4_inode {
  __le16  i_mode;    // 读写权限 
  __le16  i_uid;    // 所属用户
  __le32  i_size_lo;  // 文件大小
  __le32  i_atime;  // 最近一次的访问时间
  __le32  i_ctime;  // 最近一次更改inode时间
  __le32  i_mtime;  // 最近一次更改文件时间
  __le32  i_dtime;  // 删除时间
  __le16  i_gid;    // 所属组
  __le16  i_links_count;  //链接的数量
  __le32  i_blocks_lo;  // 占用块数
  __le32  i_flags;  // 文件标识
......
  __le32  i_block[EXT4_N_BLOCKS]; // 指向块的数组
  __le32  i_generation;  /* File version (for NFS) */
  __le32  i_file_acl_lo;  /* File ACL */
  __le32  i_size_high;
......
};
```
ext4中对于块的存储是以一颗树的形式，如下图：
![]({{site.url}}/images/blog/linux-file-1.png)
iblock是一个连续的内存，包含一个ExtendHeader和4个ExtendEntry，Entry有两种类型一种用于存放真正的数据，另一种指向其他的extend结构中的ExtendHeader，如果文件不是特别小4个Entry不够的话那么extend会裂化为一颗树，而非根节点的extend结构会包含更多的ExtendEntry。

##### inode和block在磁盘怎么存储
Inode是如何保存在磁盘上的呢，先看下图：
![]({{site.url}}/images/blog/linux-file-2.png)

1. 超级块：用于保存整个文件系统的全局信息。（并非每个块组都有）
	- s_inodes_count 整个文件系统一共有多少 inode；
	- s_blocks_count_lo 一共有多少块；
	- s_inodes_per_group 每个块组有多少 inode；
	- s_blocks_per_group 每个块组有多少块；
2. 块位图：每个块组中都有一个块位图用来记录对应块的使用情况
3. inode位图：每个块组有一个inode位图用来记录inode的使用情况
4. inode列表：真正存放inode结构数据的块
5. 数据块：真正存放文件数据的数据块
6. 块组描述符表：描述过个块组全局信息的数据块（并非每个块组都有）
7. 块组：包含上述各种数据块的一组数据块
8. 元块组：每64个块组组成一个元块组，每个元块组中的块组描述符只需要记录该元块组中块组的信息，自己本分自己的。

### Linux如何维护打开文件

现在我们知道linux的每个文件都会关联一个inode，linux想要打开一个文件只要找到对应的inode即可。
![]({{site.url}}/images/blog/linux-file-3.png)

#### 文件描述符
linux内核为每一个进场维护了一个task_struct结构，用于保存进程相关的信息，其中有一项files（结构是files_struct）,files里面有个一数组叫做fd_array，该数组的下标其实就是我们所说的文件描述符，而该数组的内容则是一个指针指向了一个file的结构，该结构记录了打开文件的信息，其中就保存有inode的对应信息。

#### 打开文件表
上面说的打开文件的file结构体实际上在linux内核中是保存在一个数组里，就是我们所说的系统级的打开文件表。

#### 分配fd方式
进程每打开一个文件就会从fd_array中找到一个未使用的最小的位置，在里面初始化一个指向file结构的指针


