# Hadoop面经

jess  



---

- ## Hadoop1架构
### 1. HDFS  
&emsp;&emsp;分布式文件存储系统  
&emsp;&emsp;进程名称：DataNode

### 2. MapReduceV1
&emsp;&emsp;计算框架  
&emsp;&emsp;进程名称：TaskTracker、JobTracker

---

- ## Hadoop2架构  
### 1. HDFS分布式文件存储系统 
### 1.1 进程：Datanode、Namenode、SecondaryNamenode(standby)

### 1.2 Namenode  
&emsp;&emsp;管理文件命名空间(NameSpace)。它维护着文件系统树(filesystem tree)以及文件树中所有的文件和文件夹的元数据(metadata)。维护这些信息的文件有两个，分别是NameSpace镜像文件(Namespace image)和操做日志文件(edit log)，这些信息被Cache在RAM中，当然，这两个文件也会被持久化存储在本地硬盘。Namenode记录着每个文件中各个块所在的数据节点的位置信息，但是他并不持久化存储这些信息，因为这些信息会在系统启动时从数据节点重建。

### 1.3 Datanode
&emsp;&emsp;Datanode是文件系统的工作节点，他们根据客户端或者是namenode的调度存储和检索数据，并且定期向namenode发送他们所存储的块(block)的列表。  
&emsp;&emsp;集群中的每个服务器都运行一个DataNode后台程序，这个后台程序负责把HDFS数据块读写到本地的文件系统。当需要通过客户端读/写某个 数据时，先由NameNode告诉客户端去哪个DataNode进行具体的读/写操作，然后，客户端直接与这个DataNode服务器上的后台程序进行通 信，并且对相关的数据块进行读/写操作。

### 1.4 SecondaryNamenode(Namenode快照)
&emsp;&emsp;Secondary  NameNode是一个用来监控HDFS状态的辅助后台程序。就想NameNode一样，每个集群都有一个Secondary  NameNode，并且部署在一个单独的服务器上。Secondary  NameNode不同于NameNode，它不接受或者记录任何实时的数据变化，但是，它会与NameNode进行通信，以便定期地保存HDFS元数据的 快照。由于NameNode是单点的，通过Secondary  NameNode的快照功能，可以将NameNode的宕机时间和数据损失降低到最小。同时，如果NameNode发生问题，Secondary  NameNode可以及时地作为备用NameNode使用。

### 1.5 Namenode容错机制
&emsp;&emsp;第一种方式是将持久化存储在本地硬盘的文件系统元数据备份。Hadoop可以通过配置来让Namenode将他的持久化状态文件写到不同的文件系统中。这种写操作是同步并且是原子化的。比较常见的配置是在将持久化状态写到本地硬盘的同时，也写入到一个远程挂载的网络文件系统。

&emsp;&emsp;第二种方式是运行一个辅助的Namenode(Secondary Namenode)。 事实上Secondary Namenode并不能被用作Namenode它的主要作用是定期的将Namespace镜像与操作日志文件(edit log)合并，以防止操作日志文件(edit log)变得过大。通常，Secondary Namenode 运行在一个单独的物理机上，因为合并操作需要占用大量的CPU时间以及和Namenode相当的内存。辅助Namenode保存着合并后的Namespace镜像的一个备份，万一哪天Namenode宕机了，这个备份就可以用上了。

&emsp;&emsp;但是辅助Namenode总是落后于主Namenode，所以在Namenode宕机时，数据丢失是不可避免的。在这种情况下，一般的，要结合第一种方式中提到的远程挂载的网络文件系统(NFS)中的Namenode的元数据文件来使用，把NFS中的Namenode元数据文件，拷贝到辅助Namenode，并把辅助Namenode作为主Namenode来运行。

(D:\迅雷下载\210922hbjbhxwftf3dzass.png)

### 2. Yarn(MapReduceV2)第二代计算框架
 
### 2.1 进程：Nodemanager  

---

- ## wordcount流程

---

- ## shuffle partition combine 过程

---

- ## map reduce数量设计

---

- ## zookeeper工作原理、选举原理
  
  选出编号最小的作为leader，其他作为follower；集群资源集中组织成树状结构

---

- ## hive hql，有没有用过hive、用hive来做什么

---

- ## hbase 框架、工作原理、存储原理、常用shell命令

---

- ## 算法的鲁棒性、可扩展性

---

- ## kmeans收敛、过程

---

- ## 神经网络常用激活函数，singmoid、tanh、relu

---

- ## 梯度下降法

---

- ## 准确率、召回率、f1值

---

- ## 数据预处理
  
    - 集成
    - 变换
    - 维度规约
    - 数值规约

---