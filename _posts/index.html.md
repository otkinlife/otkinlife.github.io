---
layout: post
title: 基本算法：二分查找
date: 2021-02-19
tags: ["基础算法","算法"]
categories: 算法

---

<!-- wp:paragraph -->

二分查找基于有序列表快速定位元素位置，核心思想是找到元素位置的中位数与查找的key作比较，从而快速缩小查找的范围

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

复杂度为O(logn)

<!-- /wp:paragraph -->

```go

func search(list []int, key int, start int, end int) int {
        //获取查找的范围长度
	length := end - start + 1
	if length <= 0 {
		return -1
	}
        // 获取中间位置
	mid := start + length/2
	if key == list[mid] {
		return mid
	}
	if key > list[mid] {
		return search(list, key, mid + 1, end)
	}
	if key < list[mid] {
		return search(list, key, start, mid - 1)
	}
	return -1
}

```