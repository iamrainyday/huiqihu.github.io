## Reading Lists


### Distributed Shared Memory (DSM)

 分布式共享内存是分布式系统中一个有一定历史的研究话题。 除提供共享内存的功能外，其研究核心可以归结为使用复制技术提升读/写操作的本地性(locality)，包括复制的粒度，副本的跟踪(不是所有其他进程都包含副本)，副本一致性(实现不同的内存一致性模型)。 值得注意的是，体系结构里面多核处理器共享内存也面临着相同问题，所以DSM对体系结构里很多协议都多有借鉴。
 
Memory consistence models. Here's a very clear introduction. 
 
近年来，数据库方面研究也有相近的研究。 典型的应用就是分布式KVS系统，将复制对象定义为item，缓存item以提高系统吞吐。如下面的两篇论问能看到体系结构一致性协议(e.g. MESI协议)的影子。
 
* EuroSys-2018,  [Scale-Outcc NUMA: Exploiting Skew with Strongly Consistent Caching](http://homepages.inf.ed.ac.uk/s1372211/pub/eurosys18.pdf) 

* VLDB-18, [Efficient Distributed Memory Management with RDMA and
Caching](http://www.vldb.org/pvldb/vol11/p1604-cai.pdf)


### RDMA based system

* RDMA通讯方式简介，[a brief introduction of RMDA](rdma_introduction.md)

* 基于RDMA的KVS系统， [a brief survey of RDMA based KVS](rdma_kvs.md)









