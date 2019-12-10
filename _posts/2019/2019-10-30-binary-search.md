---
layout: post
title: 数据结构与算法之美-二分查找
category: note
tags: [algorithm]
keywords: algorithm
no-post-nav: true
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

## 什么是二分查找
二分查找是一种非常简单易懂的快速查找算法，针对一个**有序**的数据集合，查找思想类似分治的思想。每次都是通过和中间元素的比较，将待查找区间缩小为之前的一半，直至找到目标元素或待查找区间被缩小为0。

## 复杂度分析
- 查找过程中只需要有记录区间左、右边界的变量即可，所以空间时间复杂度为O(1)。
- 根据二分查找的思想，每次查找都会将待查找区间缩小为之前的一半，所以时间复杂度为O($\log^n$)。

## 代码实现
二分查找的思想很容易理解，但是写好一个没有bug的二分查找代码还是有点难度的，特别是一些二分查找的变体实现。例如查找第一个目标元素、查找最后一个目标元素、查找第一个大于等于目标元素的元素、查找最后一个小于等于目标元素的元素。[点击这里](https://github.com/wyc18556/algorithms/blob/master/src/other/BinSearch.java)查看一般的二分查找实现，[点击这里](https://github.com/wyc18556/algorithms/blob/master/src/other/BinSearchPlus.java)查看变体的二分查找实现。

## 局限性
- 依赖顺序表结构，即数组。
- 针对的只能是有序的数据集合。
- 数据量太小没必要使用二分查找，顺序遍历就足够了。
- 由于底层依赖数组的原因，要求内存空间连续，如果要查找的数据量很大，就会对内存空间的要求比较苛刻。