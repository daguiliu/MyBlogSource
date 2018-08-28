---
title: springboot服务shell启动脚本
date: 2018-08-28 10:13:39
tags:
- linux
- shell
- netstat
- springboot
categories: 编程
---
linux环境下，先新建xxxx.sh文件，下面是脚本内容
```java
#!/bin/sh

netstat -anp | grep java | grep 1511|awk '{print $7}'|cut -d / -f 1|xargs kill -9
netstat -anp | grep java | grep 1512|awk '{print $7}'|cut -d / -f 1|xargs kill -9

rm -f tppid

nohup  /opt/APP/java/bin/java    -jar /opt/APP/feeder-1.0-SNAPSHOT.jar  --spring.config.location=classpath:/application-test.yml  --server.port=1511 > /dev/null 2>&1 &

nohup  /opt/APP/java/bin/java    -jar /opt/APP/feeder-1.0-SNAPSHOT.jar  --spring.config.location=classpath:/application-test-1.yml  --server.port=1512 > /dev/null 2>&1 &

echo $! > tppid

echo Start Success!

```
## 一、1>/dev/null 2>&1 含义

/dev/null ：代表空设备文件
* \>  ：代表重定向到哪里，例如：echo "123" > /home/123.txt
* 1  ：表示stdout标准输出，系统默认值是1，所以">/dev/null"等同于"1>/dev/null"
* 2  ：表示stderr标准错误
* &  ：表示等同于的意思，2>&1，表示2的输出重定向等同于1

1 > /dev/null 2>&1 语句含义：

1 > /dev/null ： 首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。
2>&1 ：接着，标准错误输出重定向（等同于）标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。


## 二、shell中$0,$?,$!等的特殊用法
* $$ Shell本身的PID（ProcessID）
* $! Shell最后运行的后台Process的PID
* $? 最后运行的命令的结束代码（返回值）
* $- 使用Set命令设定的Flag一览
* $* 所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
* $@ 所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
* $# 添加到Shell的参数个数
* $0 Shell本身的文件名

## 三、netstat查询进程

其中，netstat -anp | grep java | grep 1201|awk '{print $7}'|cut -d / -f 1|xargs kill -9
查询java服务端口为1201的进程，并且杀死

* netstat -anp 是查询所有后台进程，并且不反向解析域名。
* grep java 全文匹配关键字java
* cut -d / -f 1是截取20294/java并返回20194
* xargs是多行输入会以单行输出
