---
title: hadoop学习-hdfs
date: 2016-01-09 21:14:24
categories: hadoop
tags: [hadoop,hdfs]
---

## hadoop的子项目
* Core:一套分布式文件系统以及支持Map-reduce的计算框架
* Avro:定义了一种用于支持大数据应用的数据格式，并为这种格式提供了不同的编程语言的支持（可以理解为服务器的中间件，和dubbo等rpc框架差球不多。）
* Map/Reduce:是一个使用简易的软件框架，基于它写出来的应用程序能够支行在由上千个商用机器组成的大型集群上，并以一种可靠容错的方式并行处理上T级别的数据集
* zookeeper：是高可用的和可靠的分布式协同系统
* pig：建立于hadoop Core之上为并行计算环境提供了一套数据工作流语言和执行构架
* hive：是为提供简单的数据操作而设计的下一代分布式数据仓库。它提供了简单的类似sql的语法的hiveQL语言进行数据查询
* hbase：建立于hadoop Core之上提供一个可扩展的数据库系统
* flume：一个分布式、可靠、和高可用的海量日志聚合的系统，支持在系统中定制各类数据发送方，用于收集数据
* mahout:是一套具有可扩充能力的机器数据类库
* sqoop：apache下用于RDBMS和HDFS互相导数据的工具。
## HDFS优点
--  高容错性  
  * 数据自动保存多个副本  
  * 副本丢失后，自动恢复  
--  适合批处理  
  * 移动计算而非数据  
  * 数据位置暴露给计算框架  
--  适合大数据处理  
  * GB、TB、甚至PB级数据  
  * 百万规模以上的文件数量  
  * 10K+节点  
-- 可构建在廉价机器上  
  * 通过多副本提高可靠性  
  * 提供了容错和恢复机制  
## HDFS缺点
- 低延迟数据访问  
  比如毫秒级  
  低延迟与高吞吐率  
- 小文件存取  
  占用namenode大量内存  
  寻道时间超过读取时间  
- 并发写入，文件随机修改  
  一个文件只能有一个写者  
  仅支持append  
## HDFS结构
HDFS为了做到可靠性（reliability）创建了多份数据块（data blocks）的复制（replicas），并将它们放置在服务器群的计算节点中（compute nodes），MapReduce就可以在它们所在的节点上处理这些数据了。
### NameNode
* 存储元数据
* 元数据保存在内存中
* 保存文件，block,datanode之间的映射关系
### DataNode
* 存储文件内容
* 文件内容保存在磁盘
* 维护了block id到datanode本地文件的映射关系
## HDFS运行机制
* 一个名字节点和多个数据节点
* 数据复制（冗余机制）  
 --存放的位置（机架感知策略）   
* 故障检测  
 --数据节点  
 心跳包（检测是否宕机）  
 块报告（安全模式下检测）  
 数据完整性检测（校验和比较）  
 --名字节点（日志文件，镜像文件）  
* 空间回收机制
## HDFS数据存储单元（block） 
- 文件被切分成固定大小的数据块  
  默认数据块大小为64MB，可配置  
  若文件大小不到64MB，则单独存成一个block  
- 一个文件存储方式  
  按大小被切分成若干个block，存储到不同节点上  
  默认情况下每个block都有三个副本  
- Block大小和副本数通过Client端上传文件时设置，文件上传成功后副本数可以变更，Block Size不可变更
## NameNode(NN)
- NameNode主要功能：接受客户端的读写服务
- NameNode保存metadata信息包括：  
-文件owership和permissions  
-文件包含哪些块
-Block保存在哪个DataNode(由DataNode启动时上报)
- NameNode的metadata信息在启动会加载到内存
-metadata存储到磁盘文件名为"fsimage"
-Block的位置信息不会保存到fsimage
-edits记录对metadata的操作日志
## SecondaryNameNode(SNN)
- 它不是NN的备份（但可以做备份），它的主要工作是帮助NN合并edits log,减少NN启动时间
- SNN执行合并时机
-根据配置文件设置的时间间隔fs.checkpoint.period默认3600秒
-根据配置文件设置edits log大小fs.checkpoint.size规定edits文件的最大值默认是64M

   