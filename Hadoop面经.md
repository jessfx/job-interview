# 1. Hadoop面经

jess  

<!-- TOC -->

- [1. Hadoop面经](#1-hadoop面经)
    - [1.1. Hadoop1架构](#11-hadoop1架构)
        - [1.1.1. HDFS](#111-hdfs)
        - [1.1.2. MapReduceV1](#112-mapreducev1)
    - [1.2. Hadoop2架构](#12-hadoop2架构)
        - [1.2.1. HDFS分布式文件存储系统](#121-hdfs分布式文件存储系统)
            - [1.2.1.1. 进程](#1211-进程)
            - [1.2.1.2. NameNode](#1212-namenode)
            - [1.2.1.3. DataNode](#1213-datanode)
            - [1.2.1.4. SecondaryNameNode(NameNode快照)](#1214-secondarynamenodenamenode快照)
            - [1.2.1.5. NameNode容错机制](#1215-namenode容错机制)
        - [1.2.2. Yarn(MapReduceV2)第二代计算框架](#122-yarnmapreducev2第二代计算框架)
            - [1.2.2.1. 进程](#1221-进程)
            - [1.2.2.2. ResourceManager](#1222-resourcemanager)
            - [1.2.2.3. NodeManager](#1223-nodemanager)
    - [1.3. wordcount流程](#13-wordcount流程)
    - [1.4. shuffle partition combine 过程](#14-shuffle-partition-combine-过程)
    - [1.5. map reduce数量设计](#15-map-reduce数量设计)
    - [1.6. zookeeper工作原理、选举原理](#16-zookeeper工作原理选举原理)
    - [1.7. hive热点倾斜，hive优化](#17-hive热点倾斜hive优化)
    - [1.8. hbase 框架、工作原理、存储原理、常用shell命令](#18-hbase-框架工作原理存储原理常用shell命令)
    - [1.9. 算法的鲁棒性、可扩展性](#19-算法的鲁棒性可扩展性)
    - [1.10. kmeans收敛、过程](#110-kmeans收敛过程)
    - [1.11. 神经网络常用激活函数，singmoid、tanh、relu](#111-神经网络常用激活函数singmoidtanhrelu)
    - [1.12. 梯度下降法](#112-梯度下降法)
    - [1.13. 准确率、召回率、f1值](#113-准确率召回率f1值)
    - [1.14. 数据预处理](#114-数据预处理)

<!-- /TOC -->

---

## 1.1. Hadoop1架构
### 1.1.1. HDFS  
&emsp;&emsp;分布式文件存储系统  
&emsp;&emsp;进程名称：DataNode

### 1.1.2. MapReduceV1
&emsp;&emsp;计算框架  
&emsp;&emsp;进程名称：TaskTracker、JobTracker

---

## 1.2. Hadoop2架构  
### 1.2.1. HDFS分布式文件存储系统 
#### 1.2.1.1. 进程

- DataNode
- NameNode
- SecondaryNameNode(standby)

![](/img/210922hbjbhxwftf3dzass.png)

#### 1.2.1.2. NameNode  
&emsp;&emsp;管理文件命名空间(NameSpace)。它维护着文件系统树(filesystem tree)以及文件树中所有的文件和文件夹的元数据(metadata)。维护这些信息的文件有两个，分别是NameSpace镜像文件(Namespace image)和操做日志文件(edit log)，这些信息被Cache在RAM中，当然，这两个文件也会被持久化存储在本地硬盘。NameNode记录着每个文件中各个块所在的数据节点的位置信息，但是他并不持久化存储这些信息，因为这些信息会在系统启动时从数据节点重建。

#### 1.2.1.3. DataNode
&emsp;&emsp;DataNode是文件系统的工作节点，他们根据客户端或者是nameNode的调度存储和检索数据，并且定期向nameNode发送他们所存储的块(block)的列表。  
&emsp;&emsp;集群中的每个服务器都运行一个DataNode后台程序，这个后台程序负责把HDFS数据块读写到本地的文件系统。当需要通过客户端读/写某个 数据时，先由NamNNode告诉客户端去哪个DataNode进行具体的读/写操作，然后，客户端直接与这个DataNode服务器上的后台程序进行通 信，并且对相关的数据块进行读/写操作。

#### 1.2.1.4. SecondaryNameNode(NameNode快照)
&emsp;&emsp;Secondary  NameNode是一个用来监控HDFS状态的辅助后台程序。就想NameNode一样，每个集群都有一个Secondary  NameNode，并且部署在一个单独的服务器上。Secondary  NameNode不同于NameNode，它不接受或者记录任何实时的数据变化，但是，它会与NameNode进行通信，以便定期地保存HDFS元数据的 快照。由于NameNode是单点的，通过Secondary  NameNode的快照功能，可以将NameNode的宕机时间和数据损失降低到最小。同时，如果NameNode发生问题，Secondary  NameNode可以及时地作为备用NameNode使用。

#### 1.2.1.5. NameNode容错机制
&emsp;&emsp;第一种方式是将持久化存储在本地硬盘的文件系统元数据备份。Hadoop可以通过配置来让NameNode将他的持久化状态文件写到不同的文件系统中。这种写操作是同步并且是原子化的。比较常见的配置是在将持久化状态写到本地硬盘的同时，也写入到一个远程挂载的网络文件系统。

&emsp;&emsp;第二种方式是运行一个辅助的NameNode(Secondary NameNode)。 事实上Secondary NameNode并不能被用作NameNode它的主要作用是定期的将Namespace镜像与操作日志文件(edit log)合并，以防止操作日志文件(edit log)变得过大。通常，Secondary NameNode 运行在一个单独的物理机上，因为合并操作需要占用大量的CPU时间以及和NameNode相当的内存。辅助NameNode保存着合并后的Namespace镜像的一个备份，万一哪天NameNode宕机了，这个备份就可以用上了。

&emsp;&emsp;但是辅助NameNode总是落后于主NameNode，所以在NameNode宕机时，数据丢失是不可避免的。在这种情况下，一般的，要结合第一种方式中提到的远程挂载的网络文件系统(NFS)中的NameNode的元数据文件来使用，把NFS中的NameNode元数据文件，拷贝到辅助NameNode，并把辅助NameNode作为主NameNode来运行。

### 1.2.2. Yarn(MapReduceV2)第二代计算框架
 
#### 1.2.2.1. 进程

- master：ResourceManager
- slave：NodeManager
- ApplicationMaster
- Container
- YarnRunner
- MapTask
- ReduceTask

![](/img/669905-20170420115229618-1888016161.jpg)

#### 1.2.2.2. ResourceManager

ResourceManager是全局资源管理器，负责整个系统的资源管理和分配,包括处理客户请求、启动App Master、监控namenode、资源的分配与调度。主要由两个组件组成：调度器(Resource Scheduler)和应用程序管理器(Applications Manager,ASM)

![](/img/174531ef9foerhcqiq8cg7.jpg)

1. 调度器(Resource Scheduler)

&emsp;&emsp;调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。需要注意的是，该调度器是一个“纯调度器”，它不再从事任何与具体应用程序相关的工作，比如不负责监控或者跟踪应用的执行状态等，也不负责重新启动因应用执行失败或者硬件故障而产生的失败任务，这些均交由应用程序相关的ApplicationMaster完成。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container）表示，Container是一个动态资源分配单位，它将内存、CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。此外，该调度器是一个可插拔的组件，用户可根据自己的需要设计新的调度器，YARN提供了多种直接可用的调度器，Fifo Scheduler,Capacity Scheduler,Fair Scheduler。

Fifo Scheduler  
FIFO Scheduler把应用按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。

FIFO Scheduler是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。大的应用可能会占用所有集群资源，这就导致其它应用被阻塞。

Capacity Scheduler  
而对于Capacity调度器，有一个专门的队列用来运行小任务，但是为小任务专门设置一个队列会预先占用一定的集群资源，这就导致大任务的执行时间会落后于使用FIFO调度器时的时间。

Fair Scheduler  
在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。如下图所示，当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。

需要注意的是，在下图Fair调度器中，从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终的效果就是Fair调度器即得到了高的资源利用率又能保证小任务及时完成。

![](/img/scheduler.jpg)

#### 1.2.2.3. NodeManager



---

## 1.3. wordcount流程

---

## 1.4. shuffle partition combine 过程

---

## 1.5. map reduce数量设计

---

## 1.6. zookeeper工作原理、选举原理
  
  选出编号最小的作为leader，其他作为follower；集群资源集中组织成树状结构

---

## 1.7. hive热点倾斜，hive优化

---

## 1.8. hbase 框架、工作原理、存储原理、常用shell命令

---

## 1.9. 算法的鲁棒性、可扩展性

---

## 1.10. kmeans收敛、过程

---

## 1.11. 神经网络常用激活函数，singmoid、tanh、relu

---

## 1.12. 梯度下降法

---

## 1.13. 准确率、召回率、f1值

---

## 1.14. 数据预处理
  
    - 集成
    - 变换
    - 维度规约
    - 数值规约

---