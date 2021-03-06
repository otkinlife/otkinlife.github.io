---
title: 计算机原码、反码、补码
date: 2020/4/23 20:46:25
tags: 
    - 基础知识
categories:
    - 操作系统
---
## 计算机如何进行加减法运算

> 困扰了我很久的问题，今天决心研究一下

### 什么是原码，反码，补码

（以下前提是假设计算机字长为8位）

#### 原码
一个数的二进制表达，在计算机中最高位表示正负（0为正，1为负）。 比如说6的原码是 `0000 0110` 。而-6的原码为 `1000 0110`

#### 反码
正数的反码就是他的原码。负数的反码是除去符号位（最高位）后其他位取反。比如6的反码为 `0000 0110`, 而-6 的反码为 `1111 1001`

#### 补码
正数的补码是他的原码，负数的补码是在其反码的基础上+1。如6的补码为 `0000 0110`。 而-6的补码为 `1111 1010`

### 为什么需要反码和补码
事实上原码是我们最容易理解计算的，但是对于计算机来讲要想识别符号位然后对其数值做加减运算实现会非常复杂。于是人们将符号位也参与运算，这也是反码和补码出现的意义。

#### 计算机如何将减法变为加法

```
 如果使用原码让符号位参与计算，加法是没有问题的，但是减法如：
 10 - 6 = 0000 1010 + 1000 0110 = 1001 0000 = 144。
 这个结果明显不对
 于是人们使用补码解决了上述问题，而反码其实是通过原码求补码的一个过渡码
 10 - 6 = 0111 0101 + 1111 1010 = （1）0000 0100 = 4
 
```

#### 思考一下

用补码计算的原因：
```
    在现实中我们知道
    9 -6 = 3
    9 + 4 = 13
    发现如果我们也将最高的溢出位舍弃那么 13 去掉高位 1后的值也是3，那么我们可以说-6的补码是4（10 - 6）
    其实计算机就是用的这个方法
    -6的补码在计算机中也可以这样推算出来：256 - 6 = 250 = 1111 1010
``` 
    
#### 应用一下
在LeetCode中231题，2的幂其实可以利用位运算解决，其中涉及到了负数的补码表示