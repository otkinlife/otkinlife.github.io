---
layout: post
title: Linux内核数据结构-链表
date: 2022-07-16
tags: ["Linux","Os"]
categories: 操作系统 
---


## 通用链表结构
我们通常实现链表的方式是通过在数据结构体内部添加一个指向数据的next（或者 previous）指针，使各个数据节点串联在链表中。假定我们有一个fox数据结构来描述狐狸，结构体如下：

```c
struct fox {
    unsigned long tail_length; /*尾巴长度，以厘米为单位*/
    unsigned long weight; /*重量，以千克为单位*/
    bool is_fantastic; /*这只狐狸奇妙吗？*/
};
```

通用实现链表的方式如下：

```c
struct fox {
    unsigned long tail_length; /*尾巴长度，以厘米为单位*/
    unsigned long weight; /*重量，以千克为单位*/
    bool is_fantastic; /*这只狐狸奇妙吗？*/
    struct fox *next; /*指向下一个狐狸*/
    struct fox *prev; /* 指向前一个狐狸*/
};
```

## Linux链表实现

Linux内核方式与众不同，它不是将数据结构塞入链表，而是将链表节点塞入数据结构！
在2.1内核开发系列中，首次引人了官方内核链表实现。从此内核中的所有链表现在都使用官方的链表实现了，千万不要再自己造轮子啦！
链表代码在头文件 ＜linux/list.h>中声明，其数据结构很简单：

```c
struct list_head {
    struct list_head *next
    struct list_head *prev;
}
```

那么如何正确使用这个结构使一个普通的struct变成链表呢？答案很简单：

```c
struct fox {
    unsigned long tail_length; /* 尾巴长度，以厘米为单位*/
    unsigned long weight; /* 重最，以千克为单位*/
    bool is_fantastic; /*这只狐狸奇妙吗？*/
    struct list_head list; /*所有 fox 结构体形成链表*/
};
```

fox中的list.next指向下一个元素，list.prev指向前一个元素。现在链表已经定义好了，但是显然还不够方便。因此內核又提供了一组方法来操作链表。

### 操作方法（复杂度O(1)）

|方法                                                                 |说明                                                        |
|:-------------------------------------------------------------------|:-----------------------------------------------------------|
|list\_add(struct list\_head *new, struct list\_head *head).         |加入一个新节点到链表中。                                        |
|list\_add\_tail(struct list\_head *new, struct list\_head *head).   |把节点增加到链表尾。                                           |
|list_del(struct list\_head *entry)                                  |从链表中删除一个节点。                                         |
|list\_del\_init(struct list\_head *entry)                           |从链表中删除一个节点并对其重新初始化。                            |
|list\_move(struct list\_head *list, struct list\_head *head)        |从一个链表中移除list节点并把该节点加到head节点后面。               |
|list\_move\_tail(struct list\_head *list, struct list\_head *head)) |从一个链表中移除list节点并把该节点加到head的节点前（也就是链表尾部）。|
|list\_empty(struct list\_head *head)                                |检查链表是否为空。                                             |
|list\_splice(struct list\_head *list, struct list\_head *head)      |将两个未链接的链表链接起来。                                     |
|list\_splice\_init(struct list\_head *list, struct list\_head *head)|将两个未链接的链表链接起来，并初始化了list链表。                   |

### 查找方法（复杂度O(n)）

|方法|说明|
|:--|:--|
|list\_entry(struct list\_head *entry)|获取包含entry节点的外层结构体指针。该方法复杂度O(1)。|
|list\_for\_search(struct list\_head *entry, struct list\_head *head)|在head链表中查找entry节点。|
|list\_for\_each\_entry(struct data *data, struct list\_head *head, struct list\_head *list)|在head链表李查找list节点并将外层结构地址赋值给data。|
|list\_for\_each\_entry\_reverse(struct data *data, struct list_head *head, struct list\_head *list)|与list\_for\_each\_entry()方法类似，查找顺序为倒序。|
|list\_for\_each\_entry\_safe(struct data *data, struct list\_head *next, struct list\_head *head, struct list\_head *list)|查找并删除。|
|list\_for\_each\_entry\_safe\_reverse(struct data *data, struct list\_head *next, struct list\_head *head, struct list\_head *list)|与list\_for\_each\_entry\_safe()方法类似，查找顺序为倒序。|

事实上我们通过list\_head结构发现它的指针指向的类型也都是list\_head。那么内核提供的方法是如何定位到我们需要的外层数据结构呢？
答案是使用宏container_of(), 我们可以很方便地从链表指针找到父结构中包合的任何变量。这是因为在C语言中，一个给定结构中的变量偏移在编译时地址就被ABI固定下来了。

```c
/* container_of 的实现 */
#define container_of (ptr, type, membez) ({ \
    const typeof (((type *)0)->member) *__mptr = (ptr); \
    (type *) ( (char *)__mptr - offsetof(type, member)); \
})
```

使用container_of()宏，我们看一个简单的内核函数如何返回包含list_head的父类型结构体：

```c
#define list_entry (ptr, type, member) \
    container_of (ptr, type, member)
```

依靠list_ entry()方法，内核提供了创建、操作以及其他链表管理的各种方法。这些都不需要知道list head所嵌人对象的数据结构。
正如看到的：list head本身其实并没有意——它需要被嵌人到你自己的数据结构中才能生效。

## 实践

```c
struct fox {
    unsigned long tail_length; /*尾巴长度，以厘米为单位*/
    unsigned long weight; /*重量，以千克为单位*/
    bool is_fantastic; /*这只狐狸奇妙吗？*/
    struct list_head list;/*所有fox结构体形成链表*/
}
```

链表需要在使用前初始化。因为多数元素都是动态创建的（也许这就是需要链表的原因），因此最常见的方式是在运行时初始化链表。


```c
/*定义并初始化一个名为fox_list的链表*/
static LIST_HEAD(fox_list);

/*定义链表节点*/
struct fox *red_fox;
red_fox = kmalloc(sizeof(*red_fox), GFP_KERNEL);
red_fox->tail_length = 40;
red_fox->weight = 6;
red_fox->is_fantastic = false;
INIT_LIST_HEAD(&red_fox->list);

/*将节点加到链表中*/
list_add(&red_fox->list, &fox_list);

/*查找一个节点的外层结构*/
struct fox *search_fox_res;
list_for_each_entry(search_fox_res, &fox_list, red_fox->list);
```
<br>
---
本文参考**《Linux 内核设计与实现》**第3版
