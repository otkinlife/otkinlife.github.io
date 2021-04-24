---
layout: post
title: Redis位图
date: 2021-02-04
tags: ["Redis","Redis"]
categories: Redis

---

<!-- wp:paragraph -->

Redis的位图其实并非一个新的数据结构，而是跟字符串使用了同样的结构。我们知道Redis的字符串是以byte数组组成的，我们又知道一个字节有8bit位。所以当我们在使用Redis的位图时，其实就是用了String这个类型

<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->

#### 假设我们存储了字符"h"，那么它在Redis里是怎么存储的呢？

<!-- /wp:heading -->

<!-- wp:paragraph -->

首先我们要知道"h"这个字符的ASCII码，一般编程语言都有这样的库函数可以提供转换，或者直接去网上查。而我们得到h的ASCII码对应的二进制是0110&nbsp;1000。其实这个二进制就是"h"的位图结构。我们可以知道位图本身就是二进制的表示。

<!-- /wp:paragraph -->

<!-- wp:table -->
<figure class="wp-block-table"><table><tbody><tr><td class="has-text-align-center" data-align="center">0</td><td class="has-text-align-center" data-align="center">1</td><td class="has-text-align-center" data-align="center">1</td><td class="has-text-align-center" data-align="center">0</td><td class="has-text-align-center" data-align="center">1</td><td class="has-text-align-center" data-align="center">0</td><td class="has-text-align-center" data-align="center">0</td><td class="has-text-align-center" data-align="center">0</td></tr></tbody></table><figcaption>"h"的位图表示</figcaption></figure>
<!-- /wp:table -->

<!-- wp:heading {"level":4} -->

#### 使用方式

<!-- /wp:heading -->

<!-- wp:code -->

    # 设置位图（该设置方式等同于 set testBit h）
    setbit testBit 1 1
    setbit testBit 2 1
    setbit testBit 4 1 
    # 获取位图
    getbit testBit 1
    getbit testBit 2
    # 获取位图所表示的字符
    get testBit //该结果应该是"h"

    # 获取范围内1的个数
    bitcount testBit 0 -1

    # 获取位置
    bitpos testBit 1 //获取第一个 1位

<!-- /wp:code -->

<!-- wp:heading {"level":4} -->

#### 使用场景

<!-- /wp:heading -->

<!-- wp:paragraph -->

位图在存储上本质上就是2进制数字，它的值只有0和1。那么如果我们只需要记录某些只有两个状态的数据是不是就可以使用位图呢？比如开关、是否、真假。使用位图可以大大的节省存储的资源。但是缺点也很明显，那就是可读性很差。所以是否使用位图还要根据当时的场景去考量。

<!-- /wp:paragraph -->