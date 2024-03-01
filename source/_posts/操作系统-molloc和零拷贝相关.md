---
title: 开发模板
tags:
  - 总结
  - 操作系统
categories:
  - 操作系统
#keywords: "hello 1"
cover: https://pic.imgdb.cn/item/65e21e529f345e8d0387b178.webp
toc: True
abbrlink: 12
---

# molloc
  - molloc有两种实现方式，brk和mmap。当申请内存小于128kb时使用brk，大于则使用mmap。brk申请的空间在free释放时不会真的释放，而是会放回内存池；它的好处是可以一次分配更大内存，避免频繁申请内存带来的切换开销；但是会产生内存碎片
  - mmap申请的内存空间在free时会归还给系统。但是每次mmap申请的内存第一次访问都需要触发缺页中断；并且需要进入内核，要频繁的状态切换。mmap的原理时直接将虚拟地址与内核缓冲区地址进行映射，避免内核缓冲区到用户缓冲区的copy
# 零拷贝
  - 零拷贝是说不需要CPU将数据拷贝到另一个区域。
  - 首先是DMA代替CPU将数据从磁盘缓冲区调入内核缓冲区。
  - 由于数据加工都是在内核空间完成的，因此也不需要将数据从内核缓冲区拷贝到用户缓冲区。这一步有两种方法
    - 第一种是使用mmap+write。用mmap代替read，避免copy，此时仍然需要4次状态切换以及一次缓冲区到socket的拷贝
    - 第二种是使用sendfile。用sendfile代替read和write，这样只需要两次状态切换。同时缓冲区直接通过SGDMA传到网卡，实现真正的零拷贝
# 如何实现大文件传输
  - 针对大文件传输，不能使用零拷贝，因为pagecache不会起作用，导致其他小文件无法享用pagecache，在高并发场景下会造成严重性能问题。因此采用异步IO，绕过pagecache，同时避免read的阻塞问题。
    ![](https://cdn.jsdelivr.net/gh/lovevivi121/PicGo/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240112223858.jpg)