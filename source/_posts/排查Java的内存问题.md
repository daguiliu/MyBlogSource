---
title: 排查java的内存问题
date: 2018-08-17
tags: java
categories: 编程
---
本文参考: [文章](http://www.infoq.com/cn/articles/Troubleshooting-Java-Memory-Issues?utm_campaign=rightbar_v2&utm_source=infoq&utm_medium=articles_link&utm_content=link_text)

### 核心要点
* 排查Java的内存问题可能会非常困难，但是正确的方法和适当的工具能够极大地简化这一过程
* Java HotSpot JVM会报告各种OutOfMemoryError信息，清晰地理解这些错误信息非常重要，在我们的工具箱中有各种诊断和排查问题的工具，它们能够帮助我们诊断并找到这些问题的根本原因；
* 在本文中，我们会介绍各种诊断工具，在解决内存问题的时候，它们是非常有用的，包括：
```java
1、HeapDumpOnOutOfMemoryError和PrintClassHistogram JVM选项
2、Eclipse MAT
3、Java VisualVM
4、JConsole
5、jhat
6、YourKit
7、jmap
8、jcmd
9、Java Flight Recorder和Java Mission Control
10、GC Logs
11、NMT
12、原生内存泄露探测工具，比如dbx、libumem、valgrind和purify等。
```