# Linux中可清除的内存分配

[Purgeable Memory Allocations for Linux](https://nullprogram.com/blog/2019/12/29/)

我看了一些视频，[OS hacking:Purgeable memory](https://www.youtube.com/watch?v=9l0nWEUpg7s)，Andreas Kling的分享，他写了一个叫[Serenity](https://github.com/SerenityOS/serenity)的开源的操作系统并把过程记录分享到了视频中。在视频中他实现了可净化的内存[found on some Apple platforms](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/CachingandPurgeableMemory.html)，在内核中增加了特别的支持。一个进程告诉内核的详细范围不重要，因此系统在内存压力大时内核可以再回收--内存是可清楚的。

Linux有[madvise(2)](http://man7.org/linux/man-pages/man2/madvise.2.html)的机制，允许进程向内核提供有关内存预期如何使用的提示。感兴趣的标识是MADV_FREE：

> 应用程序不在需要在addr和len指定的范围内的页。内核因此可以释放那些页，但是会有延迟直到内存压力出现了。对于每个已经标记为释放但尚未释放的内存页，如果调用者写入页，则取消释放操作。
