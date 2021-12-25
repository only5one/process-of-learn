# ElasticSearch安装与介绍

## Elastic Stack简介

如果你没有听说过Elastic Stack，那你一定听说过ELK，实际上ELK是三款软件的简称，分别是Elasticsearch、
Logstash、Kibana组成，在发展的过程中，又有新成员Beats的加入，所以就形成了Elastic Stack。所以说，ELK是旧的称呼，Elastic Stack是新的名字。

![img.png](../../../day07/images/img.png)

全系的Elastic Stack技术栈包括：

![img_1.png](../../../day07/images/img_1.png)

### Elasticsearch

Elasticsearch 基于java，是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

### Logstash

Logstash 基于java，是一个开源的用于收集,分析和存储日志的工具。

### Kibana

Kibana 基于nodejs，也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的Web 界面，可以汇总、分析和搜索重要数据日志。

### Beats

Beats是elastic公司开源的一款采集系统监控数据的代理agent，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给Elasticsearch或者通过Logstash发送给Elasticsearch，然后进行后续的数据分析活动。Beats由如下组成:

- Packetbeat：是一个网络数据包分析器，用于监控、收集网络流量信息，Packetbeat嗅探服务器之间的流量，解析应用层协议，并关联到消息的处理，其支 持ICMP (v4 and v6)、DNS、HTTP、Mysql、PostgreSQL、Redis、MongoDB、Memcache等协议；
- Filebeat：用于监控、收集服务器日志文件，其已取代 logstash forwarder；
- Metricbeat：可定期获取外部系统的监控指标信息，其可以监控、收集 Apache、HAProxy、MongoDB
  MySQL、Nginx、PostgreSQL、Redis、System、Zookeeper等服务；

> Beats和Logstash其实都可以进行数据的采集，但是目前主流的是使用Beats进行数据采集，然后使用 Logstash进行数据的分割处理等，早期没有Beats的时候，使用的就是Logstash进行数据的采集。

## ElasticSearch快速入门

### 简介

官网：https://www.elastic.co/

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

我们建立一个网站或应用程序，并要添加搜索功能，但是想要完成搜索工作的创建是非常困难的。我们希望搜索解决方案要运行速度快，我们希望能有一个零配置和一个完全免费的搜索模式，我们希望能够简单地使用JSON通过HTTP来索引数据，我们希望我们的搜索服务器始终可用，我们希望能够从一台开始并扩展到数百台，我们要实时搜索，我们要简单的多租户，我们希望建立一个云的解决方案。因此我们利用Elasticsearch来解决所有这些问题及可能出现的更多其它问题。

ElasticSearch是Elastic Stack的核心，同时Elasticsearch 是一个分布式、RESTful风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。作为Elastic Stack的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

### 前言

Elasticsearch的发展是非常快速的，所以在ES5.0之前，ELK的各个版本都不统一，出现了版本号混乱的状态，所以从5.0开始，所有Elastic Stack中的项目全部统一版本号。目前最新版本是6.5.4，我们将基于这一版本进行学习。


### 下载

到官网下载：https://www.elastic.co/cn/downloads/

![img_2.png](../../../day07/images/img_2.png)

选择对应版本的数据，这里我使用的是Linux来进行安装，所以就先下载好ElasticSearch的Linux安装包

### 拉取Docker容器

因为我们需要部署在Linux下，为了以后迁移ElasticStack环境方便，我们就使用Docker来进行部署，首先我们拉取一个带有ssh的centos docker镜像

```bash
# 拉取镜像
docker pull moxi/centos_ssh
# 制作容器
docker run --privileged -d -it -h ElasticStack --name ElasticStack -p 11122:22 -p 9200:9200 -p 5601:5601 -p 9300:9300 -v /etc/localtime:/etc/localtime:ro  moxi/centos_ssh /usr/sbin/init
```

然后直接远程连接11122端口即可

### 集群安装

因为ElasticSearch不支持Root用户直接操作，因此我们需要创建一个elsearch用户

```bash
# 添加新用户
useradd elsearch
# 解压缩
tar -zxvf elasticsearch-7.9.1-linux-x86_64.tar.gz /data/

```

因为刚刚我们是使用root用户操作的，所以我们还需要更改一下/elasticsearch-7.14.0文件夹的所属，改为elsearch用户

```bash
chown elsearch:elsearch /soft/ -R
```

然后在切换成elsearch用户进行操作

```bash
# 切换用户
su - elsearch
```

然后我们就可以对我们的配置文件进行修改了
配置文件打开数
```
vim /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
  es soft memlock unlimited
  es hard memlock unlimited
```
*代表所有用户，es代表用户es可以改为实际使用的用户

修改内核参数
```
vim /etc/sysctl.conf
# 增加这样一条配置，一个进程在VMAs(虚拟内存区域)创建内存映射最大数量
vm.max_map_count=262144

sysctl -p
```
###创建data和logs目录
    mkdir -p /data/elasticsearch-7.10.1/data /data/elasticsearch-7.10.1/logs

在Elasticsearch中如果，network.host不是localhost或者127.0.0.1的话，就会认为是生产环境，会对环境的要求比较高，我们的测试环境不一定能够满足，一般情况下需要修改2处配置，如下：

```bash
# 修改jvm启动参数
vim config/jvm.options

#根据自己机器情况修改
-Xms128m 
-Xmx128m
```

然后在修改第二处的配置，这个配置要求我们到宿主机器上来进行配置

```bash
# 到宿主机上打开文件
vim /etc/sysctl.conf
# 增加这样一条配置，一个进程在VMAs(虚拟内存区域)创建内存映射最大数量
vm.max_map_count=655360
# 让配置生效
sysctl -p
```
###修改配置文件
191：
````
cluster.name: srm-es
node.name: node-1
path.data: /data/elasticsearch-7.10.1/data
path.logs: /data/elasticsearch-7.10.1/logs
network.host: 10.185.101.191
node.master: true
node.data: true
http.port: 9200
transport.tcp.port: 9300
#discovery.seed_hosts: ["10.185.101.191:9300","10.185.101.192:9300","10.185.101.193:9300"]
cluster.initial_master_nodes: ["node-1","node-2","node-3"]
discovery.zen.ping.unicast.hosts: ["10.185.101.191", "10.185.101.192","10.185.101.193"]
discovery.zen.minimum_master_nodes: 2
bootstrap.memory_lock: true
````

192：
````
cluster.name: srm-es
node.name: node-2
path.data: /data/elasticsearch-7.10.1/data
path.logs: /data/elasticsearch-7.10.1/logs
network.host: 10.185.101.192
node.master: true
node.data: true
http.port: 9200
transport.tcp.port: 9300
#discovery.seed_hosts: ["10.185.101.191:9300","10.185.101.192:9300","10.185.101.193:9300"]
cluster.initial_master_nodes: ["node-1","node-2","node-3"]
discovery.zen.ping.unicast.hosts: ["10.185.101.191", "10.185.101.192","10.185.101.193"]
discovery.zen.minimum_master_nodes: 2
bootstrap.memory_lock: true
````

193：
````
cluster.name: srm-es
node.name: node-3
path.data: /data/elasticsearch-7.10.1/data
path.logs: /data/elasticsearch-7.10.1/logs
network.host: 10.185.101.193
node.master: true
node.data: true
http.port: 9200
transport.tcp.port: 9300
#discovery.seed_hosts: ["10.185.101.191:9300","10.185.101.192:9300","10.185.101.193:9301"]
cluster.initial_master_nodes: ["node-1","node-2","node-3"]
discovery.zen.ping.unicast.hosts: ["10.185.101.191", "10.185.101.192","10.185.101.193"]
discovery.zen.minimum_master_nodes: 2
bootstrap.memory_lock: true
````

### 启动ElasticSearch

首先我们需要切换到 elsearch用户

```bash
su - elsearch
```

然后在到bin目录下，执行下面

```bash
# 进入bin目录
cd /soft/elsearch/bin
# 后台启动
./elasticsearch -d
```

启动成功后，访问下面的URL

```bash
http://192.168.13.3:9200/
```

如果出现了下面的信息，就表示已经成功启动了

![img_3](../../../day07/images/img_3.png)

如果你在启动的时候，遇到过问题，那么请参考下面的错误分析~

