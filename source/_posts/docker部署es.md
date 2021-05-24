---
title: docker部署es
date: 2021-05-20 1:13:39
tags:
- linux
- docker
- es
categories: 运维
---
1、拉取镜像
```java
docker pull elasticsearch:7.4.2
```
2、创建并启动容器
```java
docker run -d --name es -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" elasticsearch:7.4.2
```
创建时，调整内存的分配“ES_JAVA_OPTS=-Xms512m -Xmx512m”，
增加9200/9300的端口映射。

3、检测是否可以正常访问

http://192.168.1.162:9200/
![avatar](/images/02.png)

4、利用以下命令可以进入容器修改配置文件
```java
docker exec -it es /bin/bash
cd config
vi elasticsearch.yaml
```
- cluster.name：自定义集群名称；
- network.host：当前es节点绑定的ip地址；
- http.cors.enabled：是否支持跨域，默认为false；
- http.cors.allow-origin：当设置允许跨域，默认为*，表示支持所有域名，如果我们只是允许某些网站能访问，那么可以使用正则表达式。

5、安装es-head可视化插件
```java
docker pull elasticsearch-head:5
docker run -d -p 9100:9100 mobz/elasticsearch-head:5
```
访问地址：http://192.168.1.162:9100/

6、安装对应版本的kibana
```java
docker pull kibana:7.4.2
docker run -d -p 5601:5601 kibana:7.4.2
```
访问地址：http://192.168.1.162:5601
