## Jps

[官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html)参考

> jps位于jdk的bin目录下，其作用是显示当前系统的java进程情况，及其id号。 jps相当于Solaris进程工具ps。不象"pgrep java"或"ps -ef grep java"，jps并不使用应用程序名来查找JVM实例。因此，它查找所有的Java应用程序，包括即使没有使用java执行体的那种（例如，定制的启动 器）。另外，jps仅查找当前用户的Java进程，而不是当前系统中的所有进程。 



## 位置

我们知道，很多`Java`命令都在`jdk`的JAVA_HOME/bin/目录下面，`jps`也不例外，他就在bin目录下，所以，他是java自带的一个命令。 

## 功能

`jps`(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有`java`进程`pid`的命令，简单实用，非常适合在`linux/unix`平台上简单察看当前java进程的一些简单情况。 

## 原理

jdk中的jps命令可以显示当前运行的java进程以及相关参数，它的实现机制如下： java程序在启动以后，会在`java.io.tmpdir`指定的目录下，就是临时文件夹里，生成一个类似于`hsperfdata_User`的文件夹，这个文件夹里（在Linux中为/tmp/hsperfdata_{userName}/），有几个文件，名字就是java进程的pid，因此列出当前运行的java进程，只是把这个目录里的文件名列一下而已。 至于系统的参数什么，就可以解析这几个文件获得。 

~~~sh
[root@localhost hsperfdata_root]# pwd
/tmp/hsperfdata_root
[root@localhost hsperfdata_root]# ll
总用量 32
-rw-------. 1 root root 32768 8月  24 13:33 1450
[root@localhost hsperfdata_root]# 
~~~

上面的内容就是我机器中/tmp/hsperfdata_root目录下的内容，其中2679就是我机器上当前运行中的java的进程的pid，我们执行jps验证一下：

~~~sh
[root@localhost hsperfdata_root]# jps
1450 Bootstrap
3374 Jps
~~~

 执行了jps命令之后，我们发现有两个java进程，一个是pid为1450的tomcat运行的进程，另外一个是pid为3374的jps使用的进程（他也是java命令，也要开一个进程） 

## 使用

学习一个命令，先来看看帮助，使用`jps -help`查看帮助： 

~~~sh
[root@localhost hsperfdata_root]# jps --help
illegal argument: --help
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
~~~

**-q 只显示pid，不显示class名称,jar文件名和传递给main 方法的参数**

```
root@hos:/tmp/hsperfdata_root$ jps -q
2679
11421
```

**-m 输出传递给main 方法的参数，在嵌入式jvm上可能是null，** 在这里，在启动main方法的时候，我给String[] args传递两个参数。hollis,chuang,执行`jsp -m`:

```
root@hos:/tmp/hsperfdata_root$ jps -m
12062 JpsDemo hollis,chuang
```

**-l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名**

```
root@hos:/tmp/hsperfdata_root$ jps -l
12356 sun.tools.jps.Jps
2679 /home/hollis/tools/eclipse//plugins/org.eclipse.equinox.launcher_1.3.0.v20130327-1440.jar
12329 com.JavaCommand.JpsDemo
```

**-v 输出传递给JVM的参数** 在这里，在启动main方法的时候，我给jvm传递一个参数：-Dfile.encoding=UTF-8,执行`jps -v`：

```
root@hos:/tmp/hsperfdata_root$ jps -v
2679 org.eclipse.equinox.launcher_1.3.0.v20130327-1440.jar -Djava.library.path=/usr/lib/jni:/usr/lib/x86_64-linux-gnu/jni -Dosgi.requiredJavaVersion=1.6 -XX:MaxPermSize=256m -Xms40m -Xmx512m
13157 Jps -Denv.class.path=/home/hollis/tools/java/jdk1.7.0_71/lib:/home/hollis/tools/java/jdk1.7.0_71/jre/lib: -Dapplication.home=/home/hollis/tools/java/jdk1.7.0_71 -Xms8m
13083 JpsDemo -Dfile.encoding=UTF-8
```

PS:jps命令有个地方很不好，似乎只能显示当前用户的java进程，要显示其他用户的还是只能用unix/linux的ps命令。

> **jps是我最常用的java命令。使用jps可以查看当前有哪些Java进程处于运行状态。如果我运行了一个web应用（使用tomcat、jboss、jetty等启动）的时候，我就可以使用jps查看启动情况。有的时候我想知道这个应用的日志会输出到哪里，或者启动的时候使用了哪些javaagent，那么我可以使用jps -v 查看进程的jvm参数情况。**

## JPS失效处理

**现象：** 用ps -ef|grep java能看到启动的java进程，但是用jps查看却不存在该进程的id。待会儿解释过之后就能知道在该情况下，jconsole、jvisualvm可能无法监控该进程，其他java自带工具也可能无法使用

**分析：** jps、jconsole、jvisualvm等工具的数据来源就是这个文件（/tmp/hsperfdata_userName/pid)。所以当该文件不存在或是无法读取时就会出现jps无法查看该进程号，jconsole无法监控等问题

**原因：**

（1）、磁盘读写、目录权限问题 若该用户没有权限写/tmp目录或是磁盘已满，则无法创建/tmp/hsperfdata_userName/pid文件。或该文件已经生成，但用户没有读权限

（2）、临时文件丢失，被删除或是定期清理 对于linux机器，一般都会存在定时任务对临时文件夹进行清理，导致/tmp目录被清空。这也是我第一次碰到该现象的原因。常用的可能定时删除临时目录的工具为crontab、redhat的tmpwatch、ubuntu的tmpreaper等等

这个导致的现象可能会是这样，用jconsole监控进程，发现在某一时段后进程仍然存在，但是却没有监控信息了。

（3）java进程信息文件存储地址被设置，不在/tmp目录下 上面我们在介绍时说默认会在/tmp/hsperfdata_userName目录保存进程信息，但由于以上1、2所述原因，可能导致该文件无法生成或是丢失，所以java启动时提供了参数(-Djava.io.tmpdir)，可以对这个文件的位置进行设置，而jps、jconsole都只会从/tmp目录读取，而无法从设置后的目录读物信息，这是我第二次碰到该现象的原因

## 附：

1.如何给main传递参数 在eclipse中，鼠标右键->Run As->Run Configuations->Arguments->在Program arguments中写下要传的参数值

2.如何给JVM传递参数 在eclipse中，鼠标右键->Run As->Run Configuations->Arguments->在VM arguments中写下要传的参数值（一般以-D开头）