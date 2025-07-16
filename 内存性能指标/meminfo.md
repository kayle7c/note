# meminfo

    这是meminfo中打印出来的内容

```c
(base) [root@VM-2-41-tencentos ~]# cat /proc/meminfo
MemTotal:       15866696 kB
MemFree:        13180240 kB
MemAvailable:   14688412 kB
Buffers:          140396 kB
Cached:          1261452 kB
SwapCached:            0 kB
Active:          1227004 kB
Inactive:         479544 kB
Active(anon):     408624 kB
Inactive(anon):    71260 kB
Active(file):     818380 kB
Inactive(file):   408284 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:               200 kB   //未被写回到磁盘上的数据
Writeback:             0 kB   //正在写回到磁盘上的数据
AnonPages:        282864 kB
Mapped:           119404 kB
Shmem:            175212 kB
KReclaimable:     622264 kB
Slab:             845700 kB
SReclaimable:     622264 kB
SUnreclaim:       223436 kB
KernelStack:        3808 kB
PageTables:         6080 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     7933348 kB
Committed_AS:    1195172 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       17556 kB
VmallocChunk:          0 kB
Percpu:            28512 kB
HardwareCorrupted:     0 kB
AnonHugePages:    149504 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:      374644 kB
DirectMap2M:     5654528 kB
DirectMap1G:    10485760 kB
```

## MemFree

    表示系统现在还没有使用的内存

## MemAvailable

    表示系统中可用的内存，和MemFree的区别是：MemAvailable将系统中可以回收的内存也算进去了。

    MemAvailable = MemFree + Active(file) + Inactive(file) + KReclaimable - wmark_low

## Buffer

    一般是在向块设备写数据时的缓存，在写小数据时，会将多次小数据合成一个大数据再写入，在等待过程中还没有写入块设备的数据叫做buffer

## Cached

    一般是读缓存，从硬盘中缓存了一些文件到内存中，在下一次读可以直接从内存中读到

## file&anon

    一个页如果是有映射的对应文件（程序文件，数据文件等），就是file页，如果不是（比如malloc申请的内存，进程的堆栈）就是anon页。这里统计的是所有的文件页和匿名页，包括被swap到硬盘上的页。

## Active&Inactive

    分别对应热页和冷页，如果在一段时间没有被访问过的就是冷页，反之就是热页，在内存不足时优先回收冷页。

    Active=Active(anon)+Active(file)

    Inactive=Inactive(anon)+Inactive(file)

## SReclaimable&SUnreclaim

    这两个指标都是slab中的一部分，SReclaimable是可回收的，SUnreclaim是不可回收的。如果在申请slab的时候传递的flags参数里面包含了SLAB_RECLAIM_ACCOUNT，就代表当前slab是可回收的。在回收时有两种方法：1）被动回收：当系统内存不足时就会自动回收。2）主动回收： echo 2 > /proc/sys/vm/drop_caches 就是可以直接回收slab的cache的

## Mapped

    Mapped表示被进程mmap的页面

## AnonPages

    匿名页，和文件没有关联的页，malloc申请的就是匿名页。这里的AnonPages只统计了目前还在物理内存中的匿名页，因此AnonPages 不等于 Active(anon)+Inactive(anon)。同理，在malloc之后，如果不写这片内存，AnonPages不会增加，因为并没有分配物理页面，在写的时候AnonPages才会增加。
