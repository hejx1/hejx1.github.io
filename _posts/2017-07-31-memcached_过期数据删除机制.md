---
layout: post
title: memcached 过期数据删除机制
date: 2017-07-31
tags: memcached    
---

1. 当某一个值过期后，并没有从内存删除，因此stats统计时,curr_items有其信息
2. 当某个新值去占用它的位置时，当成空chunk来占用
3. 当 get 值时，判断是否过期，如果过期返回空，并且删除其信息，curr_items就减少了

即：这个过期只是让用户看不到这个数据而已，并没有在过期的瞬间立即从内存删除。这个称为 lazy expiration，惰性失效。

好处：节省 CPU 时间和检测的成本

### LRU

如果以 122Byte 大小的 chunk 举例，122 的chunk都满了，又有新的值（长度为120byte）要加入，要挤掉谁？
memcached 此处使用的 LRU 删除机制 （操作系统的内存管理，常用 FIFO,LRU 删除）
- LRU：Least Recently Used 最近最少使用
- FIFO：first in first out

原理：当某个单元被请求时，维护一个计数器，通过计数器来判断最近谁最少被使用。

注：即使某个key是设置的永久有效期，也一样会被踢出来！

即：老数据被踢现象