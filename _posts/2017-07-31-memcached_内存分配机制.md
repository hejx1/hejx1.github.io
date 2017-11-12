---
layout: post
title: memcached 内存分配机制
date: 2017-07-31
tags: memcached    
---

### 内存的碎片化

如果用C语言直接malloc,free 来向操作系统申请和释放内存时，在不断的申请和释放过程中，形成了一些很小的内存片段，无法再利用。

这种空闲且无法再利用的内存现象，称为++内存的碎片化++

#### memcached是如何缓解内存的碎片化的？

memcached 用 Slab Allocation 机制来管理内存

### Slab Allocation

1. 首先，像一般的内存池一样,  从操作系统分配到一大块内存。

2. 将分配的内存分割成各种尺寸的块（chunk），并把尺寸相同的块分成组（chunk的集合），chunk的大小按照一定比例逐渐递增（图1）。
  ![图1 Slab Allocation的构造图](http://img.blog.csdn.net/20130510101637811)

而且，slab allocator还有重复使用已分配的内存的目的。也就是说，分配到的内存不会释放，而是重复利用。

#### Slab Allocation的主要术语

- Page  ：分配给Slab的内存空间，默认是1MB。分配给Slab之后根据slab的大小切分成chunk。
- Chunk：用于缓存记录的内存空间。
- Slab Class：特定大小的chunk的组。

### 警示

如果有 100byte 的内容要存，但122byte大小的仓库中的chunk满了，并不会寻找更大的，如144byte的仓库来存储，防止内存浪费的现象。而是把122仓库中的旧数据踢掉！