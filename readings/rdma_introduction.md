### RDMA

#### RDMA传输

在RDMA传输中，SEND/RECEIVE是双边操作，即需要通信双方的参与，并且RECEIVE要先于SEND执行，这样对方才能发送数据，当然如果对方不需要发送数据，可以不执行RECEIVE操作，因此该过程和传统通信相似，区别在于RDMA的零拷贝网络技术和内核旁路，延迟低，多用于传输短的控制消息。 

WRITE/READ是单边操作，顾名思义，读/写操作是一方在执行，在实际的通信过程中，WRITE/READ操作是由active即客户端来执行的，而passive即服务器不需要执行任何操作。RDMA WRITE操作中，由客户端把数据从本地buffer中直接push到远程QP的虚拟空间的连续内存块中（物理内存不一定连续），因此需要知道目的地址（remote_addr）和访问权限（remote_key）。 RDMA READ操作中，是客户端直接到远程的QP的虚拟空间的连续内存块中获取数据poll到本地目的buffer中，因此需要远程QP的内存地址和访问权限。单边操作多用于批量数据传输。

可以看出，在单边操作过程中，客户端需要知道远程QP的remote_addr（要读取或写入的地址）和remote_key，而这两个信息是通过SEND/REVEIVE操作来交换的，RDMA通信过程的大致流程如下：

1）初始化context，注册内存域

2）建立RDMA连接

3）通过SEND/RECEIVE操作(这个获取远程地址的方式并不是固定的)，C/S交换包含RDMA memory region key的MSG_MR消息（一般是客户端先发送）

4）通过WRITE/READ操作，进行数据传输（单边操作）

5）发送MSG_DONE消息，关闭连接

其中作单边RDMA传输的时候，需要预先注册相应的本地和远程内存区域，并提供Remote buffer virtual address  这个远程地址。关于这个地址如何获获取，有华人在美国大学发表的一篇论文交作什么MPI框架的，采用的办法是两边机器都采用同一个RDMA地址，就是说你从这边机器的某个地址复制过去的话，rdma后数据也在远程机器的同一个地址上。应该是两边都保留了同样的一个地址了的。不过这种办法也有很大的局限性的吧。  如果不采用这总办法的话，只能老老实实通过一个不是RDMA的网络通讯，先把远程地址 给传过来了。

可以想的出，这种专门传地址的网络操作是会造成网络延时等问题的，所以网络也有看到针对这个的优化论文。也有事先在client端保存一个服务器端远程地址的队列，这样client端要发RDMA write到server去的时候，就可以直接从队列里面取得一个远程机器的地址，然后发送过去，发送的时候把这个地址也一起传过去，这样发送之前就不需要通过网络要求server来提供远程地址了，server端收到之后也知道数据写到那里去了，等server端用完之后，再把这个已经释放的远程地址通过刚才那个client的ack包的形式回应client端，client端就知道这个远程地址又可以用了，就可以继续的把它加到空闲队列里面以便下次使用。 严格来说这个办法也也是需要通过网络来传送接收端的“远程地址”的，不过是把他嵌入了正常的网络通讯协议里面去了，看上去“额外”的网络通讯都被避免了，只要合理的管理好这个远程地址的空闲队列就可以了。网上也有RDAM传输协议的标准化的草案也可以参考一下，看别人怎么解决这个问题的。因为这种传地址的非rdma操作的存在，可以知道，这种RDMA还是比较适合于大量数据的网络传输的，比如scsi磁盘读写操作等。

#### RDMA操作细节

RDMA提供了基于消息队列的点对点通信，每个应用都可以直接获取自己的消息，无需操作系统和协议栈的介入。
消息服务建立在通信双方本端和远端应用之间创建的Channel-IO连接之上。当应用需要通信时，就会创建一条Channel连接，每条Channel的首尾端点是两对Queue Pairs（QP）。每对QP由Send Queue（SQ）和Receive Queue（RQ）构成，这些队列中管理着各种类型的消息。QP会被映射到应用的虚拟地址空间，使得应用直接通过它访问RNIC网卡。除了QP描述的两种基本队列之外，RDMA还提供一种队列Complete Queue（CQ），CQ用来知会用户WQ上的消息已经被处理完。 RDMA提供了一套软件传输接口，方便用户创建传输请求Work Request(WR），WR中描述了应用希望传输到Channel对端的消息内容，WR通知QP中的某个队列Work Queue(WQ)。在WQ中，用户的WR被转化为Work Queue Element（WQE）的格式，等待RNIC的异步调度解析，并从WQE指向的Buffer中拿到真正的消息发送到Channel对端。
 
  
##### RDAM单边操作 (RDMA READ)
READ和WRITE是单边操作，只需要本端明确信息的源和目的地址，远端应用不必感知此次通信，数据的读或写都通过RDMA在RNIC与应用Buffer之间完成，再由远端RNIC封装成消息返回到本端。
对于单边操作，以存储网络环境下的存储为例，数据的流程如下：
1.   首先A、B建立连接，QP已经创建并且初始化。
  
2.   数据被存档在B的buffer地址VB，注意VB应该提前注册到B的RNIC (并且它是一个Memory Region) ，并拿到返回的local key，相当于RDMA操作这块buffer的权限。
 
3.   B把数据地址VB，key封装到专用的报文传送到A，这相当于B把数据buffer的操作权交给了A。同时B在它的WQ中注册进一个WR，以用于接收数据传输的A返回的状态。

4.   A在收到B的送过来的数据VB和R_key后，RNIC会把它们连同自身存储地址VA封装到RDMA READ请求，将这个消息请求发送给B，这个过程A、B两端不需要任何软件参与，就可以将B的数据存储到A的VA虚拟地址。

5.   A在存储完成后，会向B返回整个数据传输的状态信息。
单边操作传输方式是RDMA与传统网络传输的最大不同，只需提供直接访问远程的虚拟地址，无须远程应用的参与其中，这种方式适用于批量数据传输。


##### RDMA 单边操作 (RDMA WRITE)
对于单边操作，以存储网络环境下的存储为例，数据的流程如下：
1.   首先A、B建立连接，QP已经创建并且初始化。

2.   数据remote目标存储buffer地址VB，注意VB应该提前注册到B的RNIC(并且它是一个Memory Region)，并拿到返回的local key，相当于RDMA操作这块buffer的权限。

3.   B把数据地址VB，key封装到专用的报文传送到A，这相当于B把数据buffer的操作权交给了A。同时B在它的WQ中注册进一个WR，以用于接收数据传输的A返回的状态。

4.   A在收到B的送过来的数据VB和R_key后，RNIC会把它们连同自身发送地址VA到封装RDMA WRITE请求，这个过程A、B两端不需要任何软件参与，就可以将A的数据发送到B的VB虚拟地址。


5.   A在发送数据完成后，会向B返回整个数据传输的状态信息。
单边操作传输方式是RDMA与传统网络传输的最大不同，只需提供直接访问远程的虚拟地址，无须远程应用的参与其中，这种方式适用于批量数据传输。

##### RDMA 双边操作 (RDMA SEND/RECEIVE)
 RDMA中SEND/RECEIVE是双边操作，即必须要远端的应用感知参与才能完成收发。在实际中，SEND/RECEIVE多用于连接控制类报文，而数据报文多是通过READ/WRITE来完成的。
对于双边操作为例，主机A向主机B(下面简称A、B)发送数据的流程如下：
1.   首先，A和B都要创建并初始化好各自的QP，CQ

2.   A和B分别向自己的WQ中注册WQE，对于A，WQ=SQ，WQE描述指向一个等到被发送的数据；对于B，WQ=RQ，WQE描述指向一块用于存储数据的Buffer。

3.   A的RNIC异步调度轮到A的WQE，解析到这是一个SEND消息，从Buffer中直接向B发出数据。数据流到达B的RNIC后，B的WQE被消耗，并把数据直接存储到WQE指向的存储位置。


4.  AB通信完成后，A的CQ中会产生一个完成消息CQE表示发送完成。与此同时，B的CQ中也会产生一个完成消息表示接收完成。每个WQ中WQE的处理完成都会产生一个CQE。

双边操作与传统网络的底层Buffer Pool类似，收发双方的参与过程并无差别，区别在零拷贝、Kernel Bypass，实际上对于RDMA，这是一种复杂的消息传输模式，多用于传输短的控制消息。

##### Infiniband VERBS AP如何实现WRITE/READ和SEND/RECEIVE
Infiniband VERBS API中（<infiniband/verbs.h>）的有关WRITE/READ和SEND/RECEIVE涉及到的函数如下所示

![image](069A64326B4A4093B4316920DDB3DDD1)


把WR发送到QP的SQ中，根据send_wr中的opcode区分是SEND/WRITE/READ操作

int ibv_post_send(struct ibv_qp *qp, struct ibv_send_wr *wr, struct ibv_send_wr **bad_wr);

qp是从ibv_create_qp()返回的QP，wr是要发送到QP的SQ中的wr链表，bad_wr：指向它的指针将填充第一个处理失败的工作请求。返回0表示成功。struct ibv_send_wr结构体中有个成员sg_list，指定用于读取或写入的本地内存buffer，至于是读取还是写入，取决于操作码opcode，对于SEND/WRITE操作，sg_list指定要读取的内存buffer，对于READ操作，sg_list指定要写入的内存buffer。

RECEIVE操作，recv_wr中并没有操作码opcode，因为只有一个接收操作

int ibv_post_recv(struct ibv_qp *qp, struct ibv_recv_wr *wr, struct ibv_recv_wr **bad_wr);

wr是发送到QP的RQ中的wr，bad_wr指向它的指针将会用来填充第一个失败的处理请求。返回0表示成功。

可见，RDMA_CM API中，把四种常见的操作分割开来，而在Infiniband VERBS API中，SEND/READ/WRITE都是通过ibv_post_send()实现，而RECEIVE是通过ibv_post_recv()实现。

注意，无论是send/recv还是read/write，直观上看是数据直接发送，但其实是这些操作会对应一个注册过的内存，和RDMA直接交互，由RDMA直接读取和写入数据，避免应用内存和内核内存进行数据拷贝带来的开销。比如send，发送的数据所在内存是已经注册过的，RNIC异步调度轮到相应的WQE的时候，发现是SEND操作，就会直接和注册过的内存交互，把数据发送出去。比如recv，接收的缓冲区是注册过的，当RNIC异步调度轮到对应的WQE的时候，发现是RECV操作，RDMA就会直接把数据存到相应的地址中。比如write，发送的数据所在的内存是已经注册过的，RNIC直接和这块内存交互，把数据写到服务器端的目的地址。比如read，用于存储数据的接收内存是已经注册过的，RNIC直接通过网络交互，把服务器的数据直接拿过来，存到接收内存。

相当于说，通信过程需要操作的缓冲区都是注册过的，意味着可以和RDMA设备即RNIC进行直接数据交互，这样就提高了通信效率，降低延迟。send/recv和read/write操作中，都是RNIC直接和注册过的内存进行数据交互，低延迟、高吞吐量、低CPU占用率。

