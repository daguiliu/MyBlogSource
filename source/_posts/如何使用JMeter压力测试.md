---
title: 如何使用JMeter压力测试
date: 2018-08-07
tags: jmeter
categories: 编程
---

### 1、启动jmeter

运行bin/jmeter.bat

### 2、添加线程组

在TestPlan节点上右键，Add-->Threads(Users)-->Thread Group。

![avatar](/images/threadGroup.png)

Number of Threads (Users)：要模拟的并发用户量。

Ramp Up Period (in seconds)：在多长时间内均匀启动所有的线程。比如Number of Threads设为10，Ramp Up Period设为1，则jmeter每隔0.1秒启动1个线程。

Loop Count：单用户任务重复执行的次数。可以设为Forever，这样jmeter就不会自动停止，需要强制终止。

还可以设置Scheduler Configuration。这里有两组设置：指定StartTime和End Time让jmeter在特定的时间区段内执行工作；Startup Delay表示从当前时刻开始延迟多长时间开始运行，Duration设定运行时长。

### 3、设置请求头HTTP Header manager
在Thread Group上右击，Add-->Config Element-->HTTP Header manager

Method：请求方式 post或者get

path：请求接口名

### 4、设置请求默认值HTTP Request Defaults
在Thread Group上右击，Add-->Config Element-->HTTP Request Defaults

Server Name or IP：服务器地址

port Number：端口

### 5、用jmetr向服务器发送HTTP Request
在Thread Group上右击，Add-->Sampler-->HTTP Request

上面步骤4添加了请求值，这里就不需要填写服务器地址

Method：请求方式 post或者get

path：请求接口名

### 6、添加Listener

在TestPlan上右击，Add-->Listener-->View Result Tree。可以查看请求结果。

在TestPlan上右击，Add-->Listener-->Aggregate Report。可以查看jmeter生成的报告，Jmeter生成的报告有多种。
