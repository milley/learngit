# Linux中可清除的内存分配

[Purgeable Memory Allocations for Linux](https://nullprogram.com/blog/2019/12/29/)

我看了一些视频，[OS hacking:Purgeable memory](https://www.youtube.com/watch?v=9l0nWEUpg7s)，Andreas Kling的分享，他写了一个叫[Serenity](https://github.com/SerenityOS/serenity)的开源的操作系统并把过程记录分享到了视频中。在视频中他实现了可净化的内存[found on some Apple platforms](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/CachingandPurgeableMemory.html)，在内核中增加了特别的支持。一个进程告诉内核的详细范围不重要，因此系统在内存压力大时内核可以再回收--内存是可清楚的。

Linux有[madvise(2)](http://man7.org/linux/man-pages/man2/madvise.2.html)的机制，允许进程向内核提供有关内存预期如何使用的提示。感兴趣的标识是MADV_FREE：

> 应用程序不在需要在addr和len指定的范围内的页。内核因此可以释放那些页，但是会有延迟直到内存压力出现了。对于每个已经标记为释放但尚未释放的内存页，如果调用者写入页，则取消释放操作。

鉴于这些，我在MADV_FREE上构建了概念验证/玩具，为linux提供此功能：

[https://github.com/skeeto/purgeable](https://github.com/skeeto/purgeable)

我使用mmap(2)分配匿名页。当分配单元是未加锁的--比如进程没有使用到它--它的页会被标记为MADV_FREE因此内核可以在任何时候回收它们。加锁分配单元进程可以安全的使用它们，MADV_FREE会被取消。这一切都比听起来复杂一些，这也是本文的主题。

注意：那也有MADV_DONTNEED似乎符合要求，但它在linux中的实现不正确。它会立即释放那些页，因此对于实现可清除内存是无用的。

## 可清除API

在深入研究实现之前，这是API。它是四个没有结果定义的函数。被API使用的指针正是内存分配器自身。与该指针相关的所有统计都隐藏在API消费者看不到的地方。全部文档在purgeable.h中。

```c
void *purgeable_alloc(size_t);
void  purgeable_unlock(void *);
void *purgeable_lock(void *);
void  purgeable_free(void *);
```

语法非常类似C++中的weak_ptr，因为锁定会验证分配是否仍然可用，并为其创建一个强引用，防止被擦除。虽然不像弱引用，分配更具粘性。它将会保持直到系统在真实的压力下，而不仅仅是当垃圾回收发生或者最后的强引用丢失后。

下面是如何使用它来存储解码的PNG数据，如果需要可以再次解压缩：

```c
uint32_t *texture = 0;
struct png *png = png_load("texture.png");
if (!png) die();

/* ... */

for (;;) {
    if (!texture) {
        texture = purgeable_alloc(png->width * png->height * 4);
        if (!texture) die();
        png_decode_rgba(png, texture);
    } else if (!purgeable_lock(texture)) {
        purgeable_free(texture);
        texture = 0;
        continue;
    }
    glTexImage2D(
        GL_TEXTURE_2D, 0,
        GL_RGBA, png->width, png->height, 0,
        GL_RGBA, GL_UNSIGNED_BYTE, texture
    );
    purgeable_unlock(texture);
    break;
}
```

内存在锁的状态下被分配，直到看起来立即被数据填充。在继续其他任务之前，应用程序应该先解锁它。清除内存必须使用purgeable_free()，即使purgeable_lock()失败了。它不止释放了簿记，而且释放了当前零页和映射本身。最初，我在失败时使用purgeable_lock()来释放可清除的内存，但我觉得这更清楚了。

## 可清除实现

最主要的挑战是内核不一定要连续处理MADV_FREE范围。它可能只回收一些页面，并按任意顺序进行。为了锁定区域，每个页必须单独处理。根据上面引用的每个页，反转MADV_FREE要求对每个页执行写入--触发页面故障或设置页面重写标志位。

判断页是否被清除的唯一方法是检查页是否填充了零。这很容易，如果我们确定页中的特定字节应该为零，但是，由于这是一个库，调用方可能只是存储这些页上的任何内容。

所以这是我的方案：解锁一个页，查看这个页的首字节。记住它是否为零。如果是零，给那个字节写入1。一旦给所有页完成此操作，使用madvise(2)标记所有MADV_FREE。

使用这种方法，库只需要跟踪每页的第一个信息位，而不用管页的内容。假设4kB页，每个32kB分配都有1字节的开销（摊销）--或大约0.003%的开销。还不算太糟！

锁定要清除内存要稍微复杂一点。同样，必须依次访问每个页面，如果清除了任何页面，则整个分配将被视为丢失。如果解锁时第一个字节非零，那么库将检查它是否仍未非零。如果解锁时第一个字节为零，则它准备将零写入该字节，该字节当前必须为非零。

无论哪种情况，MADV_FREE都需要使用写入，因此，库执行原子比较和交换(CAS)将正确的字节写入页面，即使它在非零情况下是一样的值。原子CAS是必不可少的，因为它可以确保页不会在检查和写入之间被清除，就像两者一起做的那样，是原子化的。如果每个页都有预期的第一个字节，并且每个CAS都成功，可清除内存已经成功锁定。

作为一种优化，库考虑的不仅仅是第一个字节，还包括每个页上的第一个长整数。当页包含非零值时，库的工作非常少，并且任意的8字节为零的概率要低很多。然而我想避免潜在的别名问题，尤其是这个库被嵌入的话，所以我传递了一个“与众不同”的想法。

## 统计

统计数据存储在缓冲区返回之前，作为可清除内存，从来没有被标记为MADV_FREE。假设4kB页，对于每128MB的可清除内存，库分配一个额外的匿名页来跟踪它。分配中的页数在可清除之前存储为size_t，其余部分是上述的每页位表。

```c
size_t *p = purgeable_alloc(1<<14);
size_t numpages = p[-1];
```

所以库可以很快速的从可清除的内存地址找到它。下面是示例：

```console
      ,--- p
      |
      v
----------------------------------------------
|...Z|    |    |    |    |    |    |    |    |
----------------------------------------------
 ^  ^
 |  |
 |  `--- size_t numpages
 |
 `--- bit table
```

缺点是程序中的缓冲区下溢会轻易的破坏数字页值，因为它们直接紧挨着。在清理内存之前，把它们移到第一页的开头是非常安全的，但是这会使得位表访问更加复杂。当区域锁定时，位表的内容并不重要，因此不会被下溢锁破坏。另一个想法是：在数字页旁加上校验和。将是一个简单的整形哈希。

这使得API非常灵活，因为消费者只需跟踪单个指针即可，可清除内存分配本身的地址。

## 值得使用？

我不太清除在实际程序中，特使是在要便于携带的软件中，我实际使用可清除内存的频率是多少。每个操作系统都有自己的实现，这部分库不能移植，因为它依赖于特定的Linux的接口和行为。

它也有一个不太可能的病理案例：想象一下一个程序有两个可清除内存，它们足够大其中一个总是影响另外一个。该计划将在使用每个分配时来回打斗。检测到这种情况很困难，特别是当可清除内存分配的数量增长时。

不管怎样，这是我的软件工具带的另一个工具。
