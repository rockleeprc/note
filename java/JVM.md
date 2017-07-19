

## Linux下性能监控工具

### top

* 显示系统中各个进程的资源占用情况


  $ top

  top - 22:32:11 up 12 days, 21:58,  1 user,  load average: 1.70, 1.98, 2.26
  Tasks: 231 total,   1 running, 229 sleeping,   0 stopped,   1 zombie
  %Cpu(s): 22.1 us,  2.3 sy,  0.0 ni, 75.3 id,  0.2 wa,  0.0 hi,  0.1 si,  0.0 st
  KiB Mem :  8054928 total,  1903624 free,  4306368 used,  1844936 buff/cache
  KiB Swap: 15624188 total, 14724060 free,   900128 used.  2813424 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                  
  31761 mint      20   0 3874232 532416  93340 S  78.4  6.6 809:55.00 plugin-containe                          
  5452 mint      20   0 1858528 204900  40508 S   4.7  2.5 118:41.81 cinnamon      

### vmstat
* 查看内存,交互分区,I/O操作,上下文切换,时钟中断,CPU使用情况


  # 采样一次,共计3次
  $ vmstat 1 3
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
   r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
   1  0 899464 1749512 163668 1813516    1    5    33    35    3   60 55  6 39  1  0
   0  0 899464 1748628 163668 1813548    0    0     0     0 1131 3662 23  1 76  0  0
   2  0 899464 1748132 163668 1813548    0    0     0     0 1135 3650 23  2 75  1  0
