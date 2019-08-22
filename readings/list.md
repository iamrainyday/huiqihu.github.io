## Reading Lists

### Primary reading in Dase314

[roadmap](http://note.youdao.com/noteshare?id=8295f97258144fbedf30b55216369271&sub=WEB11c0885aac4a1eb71fbd5a83e14ff4a8).

### Distributed Consistency & Memory Consistency

分布式一致性(distributed consistency)内存一致性(memory consistency) ，（暂时我认为两者比较接近）， 内存一致性在wiki上的定义 A memory consistency model is a set of rules that governs how memory systems will process memory operations (load / store) from multiple processors. 是用来规范内存操作的一系列规则，分布式一致性可认为同样是规范分布式读写操作的一系列规则。

一致性的定义 [Memory consistence models](https://people.cs.pitt.edu/~jacklange/teaching/cs2510-f15/obsolete_lectures/07.2-Consistency%20Models.pdf))。 核心在于在有副本存在的前提下，各个进程读写操作的访问顺序规范，这个访问顺序通常是指从(多个)客户端看到的操作。

注意这里的研究的一致性与分布式共识的区别(典型的如raft, Paxos等) ， 后者主要问题是如何对单个或者多个操作(顺序)在多数派达成共识。 比如，在后者的模型中如采用raft协议，如果只有leader具有读写能力，那么不会存在分布式一致性问题； 如果follower也具有一定的读功能，那么leader和follower组合在一次的这个分布式系统，其对外提供的读写操作，将面临分布式一致性问题。最简单的例子是，如果follower上保证始终读到最新的leader上的数据，那么可以认为这个系统达到了lineariablity。 

分布式一致性或者内存一致性的核心问题出现在系统的多种应用中，下面是我列举的一些应用和相关论文：

##### 分布式共享内存 Distributed Shared Memory (DSM)

 分布式共享内存是分布式系统中一个有一定历史的研究话题。 除提供共享内存的功能外，其研究核心可以归结为使用复制技术提升读/写操作的本地性(locality)，包括复制的粒度，副本的跟踪(不是所有其他进程都包含副本)，副本一致性(实现不同的内存一致性模型)。 值得注意的是，体系结构里面多核处理器共享内存也面临着相同问题，所以DSM对体系结构里很多协议都多有借鉴。
 
 * Socc 
 
 
 ##### 数据库中的应用
近年来，数据库方面研究也有借鉴的研究。 典型的应用就是分布式KVS系统，将复制对象定义为item，缓存item以提高系统吞吐。如下面的两篇论问能看到体系结构一致性协议(e.g. MESI协议)的影子。
 
* EuroSys-2018,  [Scale-Outcc NUMA: Exploiting Skew with Strongly Consistent Caching](http://homepages.inf.ed.ac.uk/s1372211/pub/eurosys18.pdf) 

* VLDB-18, [Efficient Distributed Memory Management with RDMA and
Caching](http://www.vldb.org/pvldb/vol11/p1604-cai.pdf)


### RDMA based system

* RDMA通讯方式简介，[a brief introduction of RMDA](rdma_introduction.md)

* 基于RDMA的KVS系统， [a brief survey of RDMA based KVS](rdma_kvs.md)


* Index

### Distributed consensus protocols (Raft & Paxos Varients)

* Basis 






