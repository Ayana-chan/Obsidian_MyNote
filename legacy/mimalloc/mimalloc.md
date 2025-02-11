
# 基础知识

[堆分配算法 - yooooooo - 博客园](https://www.cnblogs.com/linhaostudy/p/10632349.html)

bump-the-pointer 机制: 连续一串内存, 从开头开始分配, 使用一个指针维护空内存的开头; 每次申请size的内存时, 就将其分配在空内存的开头, 然后让指针往后移动 size. 要求内存分配规整.

free-list 机制: 维护一个数据结构, 表示不同的空闲内存的开头; 每次分配内存时, 从中取一个空闲内存项, 将数据分配在其上, 并删除该项. 当空闲与已分配内存交错时, 这个有用.

申请内存都在thread local heap (TLB, Transfer Look-aside Table)完成, 因为申请的整个过程不可能与其他线程相关.
# 论文

[microsoft.com/en-us/research/uploads/prod/2019/06/mimalloc-tr-v1.pdf](https://www.microsoft.com/en-us/research/uploads/prod/2019/06/mimalloc-tr-v1.pdf)
把内存分为64KB的众多page, 每个page都有自己的 free list.

发现page内的内存分配完后会触发**slow path**.

分配的时候pop free list, 释放的时候push **local free list**, 这样固定的分配之后必然触发slow path (称为 **temporal cadence**, 或 generic routine), 一次性处理所有的被释放的内存. 被释放的空间通过move重新放入free list, 并清空local list.

所有分配都是在线程本地完成的, 不需要锁. 但是, 任何线程都可能释放一个对象. 因为申请内存完毕之前, 地址不可能被任何别的线程拿到; 但新建完毕之后就被任意传播了.

当一个non-local free发生的时候, 就把free的对象<u>原子性</u>地塞入该page的**thread free list**. 这使得contention只发生在一个page内.

在slow path触发的时候把thread free list <u>原子性</u>地move进free list.

page size (=`2^page_shift`)有三种:
- for small objects under 8KiB the pages are 64KiB and there are 64 in a segment.
- for large objects under 512KiB there is one page that spans the whole segment.
- huge objects over 512KiB have one page of the required size.

若有指针`p`, 和segment size `S`(是2的倍数), 则`p`所属的`segment`地址(即segment起始地址)是`segment = p ^ ~S`; `p - segment`就是page的segment内偏移长度.

如果一个page被完全free, 那么这个page就是可被回收的. 可以仅仅在slow path中需要查询新的可用的page的时候去处理这些full free pages.

> 新page称作 **fresh page**.

一次 generic routine 会遍历在<u>目标 size class</u> (仅由想要分配的内存size决定) 内的所有page, **直到找到有空闲的**(仅限于未被 full list 优化的情况). 如果没有, 就新建fresh page然后在其中分配.

> 找到空闲的page并且分配后, 下次遍历就从此项开始, 一直遍历到此项的前一项, 相当于rotate了pages数组.

由于每次都要线性遍历page, 因此如果page基本都是满的, 那么就会每次分配都遍历特别多次, 拉低性能. 解决方案是把满的page扔出去放进**full list**来独立管理, 这样剩下的page都是有空的, 不需要多遍历. 当一个满page里的内存<u>被free</u>时(不需要等generic routine), 就从full list<u>挪回来</u>.

此时 non-local free 就需要能够影响page的状态, 让其能被从full list抽出. 这使得内存申请也要原子化, 大大拖慢性能. 于是, 使用 a heap-owned list of **thread delayed free blocks** 存储所有满page的 non-local free 操作 (这种方式使得跨线程释放的pages要等generic routine才能被认为是不满的). 
- 一个未满的page标记为NORMAL, 此时 non-local free 操作放在 thread free list里面.
- 满的page标记为DELAYED, 声明此pages被放入full list, 因此此时 non-local free 操作放在 thread delayed free blocks里面.
- DELAYING标记了中间态, 保证一致性.


# C实现

[mimalloc C语言实现](https://github.com/microsoft/mimalloc)

tld: Thread local data

size class 也叫 bin.

```c
typedef enum mi_page_kind_e {  
  MI_PAGE_SMALL,    // small blocks go into 64KiB pages inside a segment  
  MI_PAGE_MEDIUM,   // medium blocks go into 512KiB pages inside a segment  
  MI_PAGE_LARGE,    // larger blocks go into a single page spanning a whole segment  
  MI_PAGE_HUGE      // a huge page is a single page in a segment of variable size (but still 2MiB aligned)  
                    // used for blocks `> MI_LARGE_OBJ_SIZE_MAX` or an aligment `> MI_BLOCK_ALIGNMENT_MAX`.} mi_page_kind_t;
```







