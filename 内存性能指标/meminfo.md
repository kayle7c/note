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

# zoneinfo

    zoneinfo里列出了各node的各个zone的相关信息，以zone normal为例

```c
Node 0, zone   Normal
  pages free     3100643   //空闲页面数
        min      15086     //最低水位线
        low      18857     //低水位线
        high     22628     //高水位线
        spanned  3604480
        present  3604480
        managed  3532725
        protection: (0, 0, 0, 0, 0)
      nr_free_pages 3100643
      nr_zone_inactive_anon 4401
      nr_zone_active_anon 74221
      nr_zone_inactive_file 164516
      nr_zone_active_file 108820
      nr_zone_unevictable 0
      nr_zone_write_pending 39
      nr_mlock     0
      nr_page_table_pages 1445
      nr_kernel_stack 3776
      nr_bounce    0
      nr_zspages   0
      nr_free_cma  0
      numa_hit     73064027
      numa_miss    0
      numa_foreign 0
      numa_interleave 24745
      numa_local   73064027
      numa_other   0
  pagesets
    cpu: 0
              count: 339
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 1
              count: 364
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 2
              count: 250
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 3
              count: 374
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 4
              count: 25
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 5
              count: 44
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 6
              count: 205
              high:  378
              batch: 63
  vm stats threshold: 64
    cpu: 7
              count: 270
              high:  378
              batch: 63
  vm stats threshold: 64
  node_unreclaimable:  0
  start_pfn:           1048576
```

+ page free：当前zone的空闲页面数（可直接分配）

+ spanned Zone：当前zone覆盖的总页面数，包含内存空洞

+ present：实际存在的物理页面 （spanned-内存空洞）

+ managed：由伙伴系统管理的页面（present-保留页）

+ min、low、high：三个水线

![](assets/249992d1aed8d8994fdc369e6103d7991a2327e1.png)

+ 当空闲内存< LOW水线时，kswapd 被唤醒，开始工作

+ 当空闲内存>HIGH水线时，kswapd 停止工作；

+ 当空闲内存<MIN水线时，OOM 触发，有进程会被 kill。

# 水线

    

```c
static void __setup_per_zone_wmarks(void)
{
    unsigned long pages_min = min_free_kbytes >> (PAGE_SHIFT - 10);
    unsigned long lowmem_pages = 0;
    struct zone *zone;
    unsigned long flags;

    /* Calculate total number of !ZONE_HIGHMEM pages */
    for_each_zone(zone) {
        if (!is_highmem(zone))
            lowmem_pages += zone_managed_pages(zone);
    }

    for_each_zone(zone) {
        u64 tmp;

        spin_lock_irqsave(&zone->lock, flags);
        tmp = (u64)pages_min * zone_managed_pages(zone);
        do_div(tmp, lowmem_pages);
        if (is_highmem(zone)) {
            /*
             * __GFP_HIGH and PF_MEMALLOC allocations usually don't
             * need highmem pages, so cap pages_min to a small
             * value here.
             *
             * The WMARK_HIGH-WMARK_LOW and (WMARK_LOW-WMARK_MIN)
             * deltas control async page reclaim, and so should
             * not be capped for highmem.
             */
            unsigned long min_pages;

            min_pages = zone_managed_pages(zone) / 1024;
            min_pages = clamp(min_pages, SWAP_CLUSTER_MAX, 128UL);
            zone->_watermark[WMARK_MIN] = min_pages;
        } else {
            /*
             * If it's a lowmem zone, reserve a number of pages
             * proportionate to the zone's size.
             */
            zone->_watermark[WMARK_MIN] = tmp;
        }

        /*
         * Set the kswapd watermarks distance according to the
         * scale factor in proportion to available memory, but
         * ensure a minimum size on small systems.
         */
        tmp = max_t(u64, tmp >> 2,
                mult_frac(zone_managed_pages(zone),
                      watermark_scale_factor, 10000));

        zone->_watermark[WMARK_LOW]  = min_wmark_pages(zone) + tmp;
        zone->_watermark[WMARK_HIGH] = min_wmark_pages(zone) + tmp * 2;
        zone->watermark_boost = 0;

        spin_unlock_irqrestore(&zone->lock, flags);
    }

    /* update totalreserve_pages */
    calculate_totalreserve_pages();
}
```

    /proc/sys/vmmin_free_kbytes：用于设置min页面的最小字节数，会转换为对应的页面数pages_min。

    /proc/sys/watermark_scale_factor:用于设置low和high

    min=pages_min*（当前zone页数/总低内存页数）

    tmp=max(min÷ 4, 当前区域页数 × watermark_scale_factor ÷ 10000)

    low=min+tmp

    high= min+tmp*2

# slabinfo

```c
[root@VM-2-41-tencentos ~]# cat /proc/slabinfo
slabinfo - version: 2.1
# name            <active_objs> <num_objs> <objsize> <objperslab> <pagesperslab> : tunables <limit> <batchcount> <sharedfactor> : slabdata <active_slabs> <num_slabs> <sharedavail>
rpc_inode_cache       23     23    704   23    4 : tunables    0    0    0 : slabdata      1      1      0
rpc_buffers           16     16   2048   16    8 : tunables    0    0    0 : slabdata      1      1      0
rpc_tasks             32     32    256   32    2 : tunables    0    0    0 : slabdata      1      1      0
```

从左往右分别对应的是：

- **name**：Slab 缓存的名称
- **active_objs**：当前活跃的对象数量。
- **num_objs**：分配的总对象数量。
- **objsize**：每个对象的大小（以字节为单位）。
- **objperslab**：每个 Slab 缓存中包含的对象数量。
- **pagesperslab**：每个 Slab 缓存使用的页面数量。
- **limit**：缓存的限制值。
- **batchcount**：批量计数。
- **sharedfactor**：共享因子。
- **active_slabs**：当前活跃的 Slab 缓存数量。
- **num_slabs**：分配的总 Slab 缓存数量。
- **sharedavail**：共享可用的 Slab 缓存数量。
