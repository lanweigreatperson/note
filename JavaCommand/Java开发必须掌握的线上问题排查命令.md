#### Java开发必须掌握的线上问题排查命令

作为一个合格的开发人员，不仅要能写得一手还代码，还有一项很重要的技能就是排查问题。这里提到的排查问题不仅仅是在coding的过程中debug等，还包括的就是线上问题的排查。由于在生产环境中，一般没办法debug（其实有些问题，debug也白扯。。。）,所以我们需要借助一些常用命令来查看运行时的具体情况，这些运行时信息包括但不限于运行日志、异常堆栈、堆使用情况、GC情况、JVM参数情况、线程情况等。

给一个系统定位问题的时候，知识、经验是关键，数据是依据，工具是运用知识处理数据的手段。为了便于我们排查和解决问题，Sun公司为我们提供了一些常用命令。这些命令一般都是jdk/lib/tools.jar中类库的一层薄包装。随着JVM的安装一起被安装到机器中，在bin目录中。下面就来认识一下这些命令以及具体使用方式。文中涉及到的所有命令的详细信息可以参考 javaCommand中关于Jvm常用命令的学习。

## jps

### 功能

显示当前所有java进程pid的命令。

### 常用指令

`jps`：显示当前用户的所有java进程的PID

`jps -v 3331`：显示虚拟机参数

`jps -m 3331`：显示传递给main()函数的参数

`jps -l 3331`：显示主类的全路径

##### [详细介绍](../JavaCommand/jps.md)

------

## jinfo

### 功能

实时查看和调整虚拟机参数，可以显示未被显示指定的参数的默认值（`jps -v 则不能`）。

> jdk8中已经不支持该命令。

### 常用指令

`jinfo -flag CMSIniniatingOccupancyFration 1444`：查询CMSIniniatingOccupancyFration参数值

##### [详细介绍](../JavaCommand/jinfo.md)

------

## jstat

### 功能

显示进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

### 常用指令

`jstat -gc 3331 250 20` ：查询进程2764的垃圾收集情况，每250毫秒查询一次，一共查询20次。

`jstat -gccause`：额外输出上次GC原因

`jstat -calss`：件事类装载、类卸载、总空间以及所消耗的时间

##### [详细介绍](../JavaCommand/jstat.md)

------

## jmap

### 功能

生成堆转储快照（heapdump）

### 常用指令

`jmap -heap 3331`：查看java 堆（heap）使用情况

`jmap -histo 3331`：查看堆内存(histogram)中的对象数量及大小

`jmap -histo:live 3331`：JVM会先触发gc，然后再统计信息

`jmap -dump:format=b,file=heapDump 3331`：将内存使用的详细情况输出到文件，之后一般使用其他工具进行分析。

##### [详细介绍](../JavaCommand/jmap.md)

------

## jhat

### 功能

一般与jmap搭配使用，用来分析jmap生成的堆转储文件。

> 由于有很多可视化工具（Eclipse Memory Analyzer 、IBM HeapAnalyzer）可以替代，所以很少用。不过在没有可视化工具的机器上也是可用的。

### 常用指令

`jmap -dump:format=b,file=heapDump 3331` + `jhat heapDump`：解析Java堆转储文件,并启动一个 web server

##### [详细介绍](../JavaCommand/jhat.md)

------

## jstack

### 功能

生成当前时刻的线程快照。

### 常用指令

`jstack 3331`：查看线程情况

`jstack -F 3331`：正常输出不被响应时，使用该指令

`jstack -l 3331`：除堆栈外，显示关于锁的附件信息

##### [详细介绍](../JavaCommand/jstack.md)

------

## 常见问题定位过程

### 频繁GC问题或内存溢出问题

一、使用`jps`查看线程ID

二、使用`jstat -gc 3331 250 20` 查看gc情况，一般比较关注PERM区的情况，查看GC的增长情况。

三、使用`jstat -gccause`：额外输出上次GC原因

四、使用`jmap -dump:format=b,file=heapDump 3331`生成堆转储文件

五、使用jhat或者可视化工具（Eclipse Memory Analyzer 、IBM HeapAnalyzer）分析堆情况。

六、结合代码解决内存溢出或泄露问题。

------

### 死锁问题

一、使用`jps`查看线程ID

二、使用`jstack 3331`：查看线程情况

## 结语

经常使用适当的虚拟机监控和分析工具可以加快我们分析数据、定位解决问题的速度，但也要知道，工具永远都是知识技能的一层包装，没有什么工具是包治百病的。