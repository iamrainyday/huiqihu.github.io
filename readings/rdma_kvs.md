
## PILAF 2013
>Christopher Mitchell, Yifeng Geng∗, Jinyang Li  
>New York University, ∗Tsinghua University

### Goals
- Friendly to read-intensive
- Using RDMA-read
- Strong consistency

### Appproaches
-  Pilaf implement RDMA read for `get()`, and use verb messages for `put()`. 
-  Clients read data directly from the server, and the server handles writes via the queue. 
-  self-verifying data structure to detect write-read race.
-  ![img](https://note.youdao.com/yws/api/personal/file/WEB1cd214b3864e441947bbc8735b4f2ab5?method=download&shareKey=19128ef65cb143097652ae4a5d9fabb8)
- Uses a memory barrier to force any updates from the CPU cache to the main memory before replying to a put request. 


### Issues
- Scalability is not metionted.
- verb messages are not good enough.


## MICA 2014
>Hyeontaek Lim, Carnegie Mellon University;  
Dongsu Han, Korea Advanced Institute of Science and Technology (KAIST);  
David G. Andersen, Carnegie Mellon University;   
Michael Kaminsky, Intel Labs 
- **M**emory-store with **I**ntelligent **C**oncurrent **A**ccess
### GOALS
#### High single node throughput
- more able to handle load spikes and popularity hot spots
- fewer nodes can reduce job latency by reducing the number of servers
#### Low end-to-end latancy
- both local key-value processing latency and the number of round-trips
#### Optimized to write-intensive workload
- demand fast processing for write-intensive workloads
#### others
- ...
### Parallel access
#### Keyhash-Based Partitioning MICA
- EREW(Exclusive Read Exclusive Write) mode
- CREW(Concurrent Read Exclusive Write) mode
### Network request handling
#### The network stack achieves zero-copy packet I/O and request processing.
- TCP processing alone may consume **70% of CPU time** on a many-core optimized key-value store
- > Z. Metreveli, N. Zeldovich, and M. F. Kaashoek. CPHash: a cache-partitioned hash table. In Proceedings ofthe 17th ACM SIGPLAN Symposium on Principles and Practice of Parallel
- MICA uses the DPDK’s burst packet I/O to transfer multiple packets
- > DPDK使用了轮询(polling)而不是中断来处理数据包。在收到数据包时，经DPDK重载的网卡驱动不会通过中断通知CPU，而是直接将数据包存入内存，交付应用层软件通过DPDK提供的接口来直接处理，这样节省了大量的CPU中断时间和内存拷贝时间。

### Data structure
- slab memory allocator: divide memory into fixed-length classes
- append-only log structures: is uncommon in in-memory kvs.(remain some questions)
- using hugepages(2MB) in 16GB memory.

### Concurrent access
- OCC: MICA checks if the initial state of the version number of the bucket is even-numbered, and upon completion of data fetch from the index and log, it reads the version number **again to check if the final version number is equal to the initial version numbe**r.

### Some Conclusion
- Batch multiple requests may cause the problem that **large batches are unrealistic** in large cluster key-value stores because it is **more difficult to accumulate multiple requests** being sent to the same server from a single client.
- Most key-value items will fit comfortably in a **single** packet 
- >  B. Atikoglu, Y. Xu, E. Frachtenberg, S. Jiang, and M. Paleczny. Workload analysis of a large-scale key-value store. In Proceedings ofthe SIGMETRICS’12, June 2012.



## HERD 2014
>Anuj Kalia, Michael Kaminsky†, David G. Andersen   
>Carnegie Mellon University, †Intel Labs

### Goals
- Use a single round trip
- Increase throughput
- Improve scalability

### Approaches
- Request-reply with server CPU involvement +
WRITEs faster than READs
- Low level verbs optimizations
- Use datagram transport



## RDMA-HBASE 2012
>Jian Huang1, Xiangyong Ouyang1, Jithin Jose1, Md. Wasi-ur-Rahman1, Hao Wang1, Miao Luo1, Hari Subramoni1, Chet Murthy2, and Dhabaleswar K. Panda1  
1 Department of Computer Science and Engineering, The Ohio State University  
2 IBM T.J Watson Research Center, Yorktown Heights, NY


- Makes it RDMA capable.
- Can we characterize the performance of Java Sockets-based communication?
- Can we improve HBase performance by using high- performance networks such as InfiniBand？
- Will HBase show improvements in query latency and trans- action throughput? 


## RDMA-MEMCACHED 2011

> Jithin Jose, Hari Subramoni, Miao Luo, Minjia Zhang, Jian Huang, Md. Wasi-ur-Rahman, Nusrat S. Islam, Xiangyong Ouyang, Hao Wang, Sayantan Sur and Dhabaleswar K. Panda 
Department of Computer Science and Engineering, The Ohio State University

- Can Memcached be re-designed from the ground up to utilize RDMA capable networks?
- A novel communication library called Unified Communication Runtime (UCR) to enable data-center middleware.
- Redesign of Memcached using UCR to dramatically improve performance when using RDMA capable networks. 
- However，it was published in 2011.

## KV-Direct 2017
> Bojie Li*§† Zhenyuan Ruan*‡† Wencong Xiao•† Yuanwei Lu§†  
Yongqiang Xiong† Andrew Putnam† Enhong Chen§ Lintao Zhang†  
†Microsoft Research §USTC ‡UCLA •Beihang University

- Develop a programmable NIC, bypass- ing host CPU.


