

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



### iostat
* 详细I/O信息


    $ sudo apt install sysstat

    # 输出信息每1秒采样一次,合计采样2次
    $ iostat 1 2
    # -d 输出磁盘使用情况,不显示cup使用情况
    $ iostat -d 1 2
    # 更多统计信息
    $ iostat -x 1 2

## JDK监控工具


### jstat
* 用于观察Java应用程序运行时的相关工具


    # 与GC相关的堆信息
    $ jstat -gc 1938
     S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
    20992.0 20992.0  0.0   6490.1 456192.0 107415.9  70656.0    18463.5   32432.0 31617.3 4016.0 3834.0      9    0.208   1      0.074    0.282

    # 与GC相关的内存信息,和各个代的最大/小值
    $ jstat -gccapacity 1938
     NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
     41984.0 671744.0 671232.0 21504.0 22016.0 626688.0    84992.0  1343488.0    70656.0    70656.0      0.0 1077248.0  32432.0      0.0 1048576.0   4016.0     15     1

     # 最近一个GC原因,及当前GC原因
     $ jstat -gccause 1938
     S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
     0.00   1.68  41.59  35.06  97.60  95.47     17    0.229     1    0.074    0.304 Allocation Failure   No GC               

     # 新生代信息
     $ jstat -gcnew 1938
      S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
      512.0 17920.0  320.0    0.0  8  15 17920.0 493568.0 344500.5     20    0.237

      # 老年代信息
      $ jstat -gcold 1938
      MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT   
      32432.0  31655.2   4016.0   3834.0     70656.0     24770.9     21     1    0.074    0.313

      # GC回收相关信息
      $ jstat -gcutil 1938
      S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
      0.00   2.16  16.01  35.07  97.65  95.47     25    0.245     1    0.074    0.319

### jinfo
* 查看正在运行Java应用程序的扩展参数


    # 显示是否打印GC日志信息
    $ jinfo -flag PrintGCDetails 1938
    -XX:-PrintGCDetails

### jmap

* 生成Java程序的堆Dump文件,查看堆内存对象实例的统计信息


    # 生成1938进程的Java程序对象统计信息
    $ jmap -histo 1938
    $ jmap -histo 1938 >/xx/xx/log.txt

    # dump当前Java程序的堆快照
    $ jmap -dump:format=b,file=/home/mint/heap.hprof 1938
    Dumping heap to /home/mint/heap.hprof ...
    Heap dump file created

### jhat
* 分析Java应用程序堆快照内容


    # 分析heap.hprof,放问127.0.0.1:7000
    $ jhat heap.hprof
    Reading from heap.hprof...
    Dump file created Wed Jul 19 23:36:57 CST 2017
    Snapshot read, resolving...
    Resolving 921860 objects...
    Chasing references, expect 184 dots........................................................................................................................................................................................
    Eliminating duplicate references........................................................................................................................................................................................
    Snapshot resolved.
    Started HTTP server on port 7000
    Server is ready.

    # Object Query Language (OQL) query
    select file.path.value.toString() from java.io.File file

### jstack
* 查看Java应用程序的线程堆栈


  # 查看线程堆栈信息
  $ jstack -l 1938
  2017-07-19 23:52:04
  Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):

  "Attach Listener" #43 daemon prio=9 os_prio=0 tid=0x00007f4ec4001000 nid=0x1c32 waiting on condition [0x0000000000000000]
     java.lang.Thread.State: RUNNABLE

     Locked ownable synchronizers:
  	- None

  "ajp-nio-8009-Acceptor-0" #41 daemon prio=5 os_prio=0 tid=0x00007f4f00582000 nid=0x7e8 runnable [0x00007f4ea90e6000]
     java.lang.Thread.State: RUNNABLE
  	at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)

### jcmd
* JDK1.7新增的多功能工具,支持导出堆信息,查看Java进程,导出线程信息,执行GC


    # 1938进程所支持的命令
    $ jcmd 1938 help
    1938:
    The following commands are available:
    JFR.stop
    JFR.start
    JFR.dump
    JFR.check
    VM.native_memory
    VM.check_commercial_features
    VM.unlock_commercial_features
    ManagementAgent.stop
    ManagementAgent.start_local
    ManagementAgent.start
    GC.rotate_log
    Thread.print
    GC.class_stats
    GC.class_histogram
    GC.heap_dump
    GC.run_finalization
    GC.run
    VM.uptime
    VM.flags
    VM.system_properties
    VM.command_line
    VM.version
    help

    For more information about a specific command use 'help <command>'.

    # 启动时间
    $ jcmd 1938 VM.uptime
    1938:
    3322.333 s

    # 打印线程信息
    $ jcmd 1938 Thread.print

    # 系统中类的统计信息
    $ jcmd 1938 GC.class_histogram

    # dump堆内存信息
    $ jcmd 1938 GC.heap_dump /home/mint/jdk.dump
    1938:
    Heap dump file created

    # 系统properties信息
    $ jcmd 1938 VM.system_properties
    1938:
    #Thu Jul 20 00:04:01 CST 2017
    java.runtime.name=Java(TM) SE Runtime Environment
    sun.boot.library.path=/opt/jdk/jdk1.8.0_131/jre/lib/amd64
    java.vm.version=25.131-b11
    shared.loader=
    java.vm.vendor=Oracle Corporation
    java.vendor.url=http\://java.oracle.com/

    # 启动参数
    $ jcmd 1938 VM.flags
    1938:
    -XX:CICompilerCount=3 -XX:InitialHeapSize=130023424 -XX:MaxHeapSize=2063597568 -XX:MaxNewSize=687865856 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=42991616 -XX:OldSize=87031808 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseParallelGC

    # 获取性能相关数据

    $ jcmd 1938 PerfCounter.print
    1938:
    java.ci.totalTime=16686437620
    java.cls.loadedClasses=5731
    java.cls.sharedLoadedClasses=0
    java.cls.sharedUnloadedClasses=0
    java.cls.unloadedClasses=22
    java.property.java.class.path="/opt/apache-tomcat-8.0.45/bin/bootstrap.jar:/opt/apache-tomcat-8.0.45/bin/tomcat-juli.jar:/opt/jdk/jdk1.8.0_131/lib/tools.jar"
    java.property.java.endorsed.dirs="/opt/apache-tomcat-8.0.45/endorsed"
    java.property.java.ext.dirs="/opt/jdk/jdk1.8.0_131/jre/lib/ext:/usr/java/packages/lib/ext"
