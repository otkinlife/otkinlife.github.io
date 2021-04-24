---
layout: post
title: 基本算法：快排
date: 2021-02-19
tags: ["基础算法","算法"]
categories: 算法

---

<!-- wp:paragraph -->

快排利用分治思想，在待排序的元素中找到一个基数。然后遍历所有元素，将比基数小的分为一组，比基数大的分为一组。然后将分好的组再次递归调用，如此最后就排好序了。

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

快排的时间复杂度为O(nlogn)，极端情况下会退化到O(n<sup>2</sup>)，快排属于原地排序算法

<!-- /wp:paragraph -->

```go
//快排的go语言实现
func main() {
	a := []int{3, 45, 1, 2, 76, 3, 6}
	QuickSort(a, 0, len(a)-1)
	fmt.Println(a)
}

//快排
func QuickSort(list []int, leftPos int, rightPos int) {
	if leftPos >= rightPos {
		return
	}
	//以最后一个元素为基数
	base := list[rightPos]
	flag := leftPos
	//从最左边开始遍历，如果比中间值小则放在左侧
	//左侧为已处理的值（比基数小），以flag作为划分
        //循环完毕后将基数与已处理后的第一个元素置换
	for i := leftPos; i < rightPos; i++ {
		if list[i] < base {
			swap(list, i, flag)
			flag++
		}
	}
	swap(list, flag, rightPos)
	QuickSort(list, leftPos, flag-1)
	QuickSort(list, flag+1, rightPos)
}

func swap(list []int, a int, b int) {
	list[a], list[b] = list[b], list[a]
}

```