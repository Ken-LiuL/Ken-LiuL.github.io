---
title: Boyer-Moore 算法
tags: 数组
categories: 算法
date: 2020-05-03 17:46:14
---


今天在做Leetcode上的[这道找到数组中的majority element](https://leetcode.com/problems/majority-element/)题目时候，发现了一个有意思的算法，Boyer-Moore算法，可以在不借助哈希表和排序的情况下，以O(n)的时间复杂度来解决这个问题。
<!-- more -->

# 常规解法

## 排序
如果我们首先将数组进行排序，然后直接返回中间的数，那么这个数字肯定是majority element。
```Golang
import "sort"

func majorityElement(nums []int) int {
    sort.Ints(nums)
    return nums[len(nums)/2]
}
```
简洁， 时间复杂度为O(NlogN)， 空间复杂度为常数项。

## 哈希
用哈希表记录下每个数字出现的次数，然后当某个数字出现的次数超过1/2的时候，就可以将其返回。
```Golang
func majorityElement(nums []int) int {
     dict := make(map[int]int)
     for _, n := range nums {
         dict[n]++
         if dict[n] > len(nums) /2 {
             return n
         }
     }
     return -1
}
```
时间复杂度为O(N)，但是空间复杂度为O(N)，并且涉及到哈希表的读取写入和扩容，实际时间并不快。

# Boyer-Moore算法

该算法可以用O(1)的空间复杂度和O(N)的时间复杂度来解决此类问题，最早在[这篇论文](http://www.cs.rug.nl/~wim/pub/whh348.pdf)被提出。我们先看看代码 
```Golang
func majorityElement(nums []int) int {
    count, res := 0, nums[0]
    for _,  n := range nums[1:] {
        if n == res {
            count++
        } else if count == 0 {
            res = n
        } else {
            count--
        }
    }
    return res
}
```
我们可以假设majority element为s， 那么数组可以视为[a...s...c...s...b..d...s...s]， 也就是各个s元素之间穿插着其他的元素，由于s的数量超过所有的元素加起来的数量，那么至少有两个s元素连在一起，那么上述的count最终肯定为正数，并且res就为我们要找的元素。
假设nums为
```Golang
[5, 5, 0, 0, 0, 5, 0, 0, 5]
```

1. res为5， count++ 一直到2
2. 遇到0，count-- 到1
3. 遇到0， count-- 到0
4. res为0， count++ 为1
5. 遇到5， count-- 到0
6. res为5， count++ 为1
7. 遇到0， count-- 到0
8. res为0， count++ 为1
9. 遇到0， count++ 到2
10. 遇到5， count-- 为1
11. 返回5作为majority element

# 总结
Boyer-Moore算法虽然并不直观，但是有良好的时间复杂度和空间复杂度，并且实现的代码非常简洁干净，多思考算法所蕴含的逻辑有助于掌握该算法。
