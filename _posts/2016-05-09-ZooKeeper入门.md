# ZooKeeper入门

---

[TOC]

## 主要功能

​    ZooKeeper是Apache软件基金会的顶级开源项目之一，致力于开发和维护一个高可用的分布式一致的开源服务器，主要的应用场景包括配置服务、命名服务和分布式同步等。
​    ZooKeeper项目提供ZooKeeper服务器和ZooKeeper客户端两个软件。服务器包括可执行的jar包和相关的启动脚本。客户端是.jar的类库，使用人员可以引用客户端类库，调用相关接口远程连接服务器，进行存取数据等操作。

​    ZooKeeper服务器一般采用集群化部署，对客户端提供如下服务：

#### 分布式一致的数据存取

ZooKeeper服务器提供树形结构的数据存取服务。用户可任意指定存储路径。每个树结点，包括内部结点和叶子结点，都可以存储数据。数据类型只有一种：字节数组。用户可执行新增、修改和删除结点3种数据操作，查询某一结点是否存在，获取某一结点的数据，以及获取某一结点的子结点路径列表。
ZooKeeper保障，在一定的网络时延内，集群中不同的可用服务器上的数据保持一致。

#### 数据变更事件的监听和通知

用户通过ZooKeeper客户端查询服务器数据时，可选择是否监听该结点的数据变更事件。事件类型包括：结点被创建事件、结点被删除事件、结点数据变更事件和结点的子结点列表变更事件。
一旦用户监听的事件发生，用户将及时收到服务器发出的事件通知。
需要注意的是，ZooKeeper的事件监听是一次性的。

#### 数据操作的顺序一致性

ZooKeeper保障服务器上数据操作的顺序一致性，以及客户端接收到的数据事件的顺序一致性。

#### 数据操作的事务性

ZooKeeper提供对服务器上数据操作的事务支持，批量提交的多个数据操作要么全部生效，要么全部失败。

#### 用户权限管理

ZooKeeper提供了结点级的数据操作的用户权限管理功能。

#### 客户端连接状态的监听和通知

ZooKeeper的3个设计间接提供了这一功能：

1. ZooKeeper服务器集群和客户端之间存在保持心跳的长连接。


1. ZooKeeper支持一种临时数据结点（Ephemeral ZNode）。此类结点无法添加自结点，并且当创建这个结点的客户端断开与服务器的长连接后，结点立即被删除。


1. 其他客户端可以通过数据变更事件监听机制收到感兴趣的客户端上线（即建立与服务器集群的长连接）和下线（即断开与服务器集群的长连接）的通知。

## 分布式一致性实现原理

#### 领导者与广播

ZooKeeper保障服务器集群的数据一致性的核心是：在任一时刻，集群中的所有数据操作本质上都来自某唯一的服务器（称为领导者，Leader），并由该服务器向其他所有服务器（称为学习者，Learner）发起数据变更的广播。
领导者和学习者之间存在保持心跳的长连接。
需要注意的是，客户端连接服务器集群时是不区分服务器的角色的。客户端向其连接的服务器发起数据变更请求后，被连接的服务器向领导者转发该数据操作，然后由领导者向其他所有服务器进行广播。

 

#### 选举机制       

ZooKeeper保障服务器集群高可用性的核心技术是领导者的选举（election）机制。
ZooKeeper服务器集群在集群启动、领导者不再可用（宕机、断网等）或者大多数跟随者不再可用时，进入选举模式。选举模式中，ZooKeeper集群不支持数据变更操作。 ​	ZooKeeper的选举算法有两种：一种是基于basic paxos实现的，另外一种是基于fast paxos算法实现的。系统默认的选举算法为fast paxos。

#### 顺序一致性

ZooKeeper上所有数据变更都是有序的。
ZooKeeper采用了递增的事务id号 （称为zxid，即ZooKeeper Transaction Id）来标识每一个数据变更操作。
实现中zxid是一个64位的数字。其中，高32位称为epoch（意为纪元）号，用 来标识leader关系是否改变，每次一个leader被选出来，它都会有一个递增的epoch号。
zxid的低32位对数据变更操作递增计数。

## 客户端API接口

本章主要内容摘录自参考文献[2]。
ZooKeeper客户端类库提供了丰富直观的API供用户程序使用，下面是一些常用的API：

1. create(path, data, flags): 创建一个ZNode, path是其路径，data是要存储在该ZNode上的数据，flags常用的有: PERSISTEN, PERSISTENT_SEQUENTAIL, EPHEMERAL, EPHEMERAL_SEQUENTAIL 。


1. delete(path, version): 删除一个ZNode，可以通过version删除指定的版本, 如果version是-1的话，表示删除所有的版本 。


1. exists(path, watch): 判断指定ZNode是否存在，并设置是否Watch这个ZNode。这里如果要设置Watcher的话，Watcher是在创建ZooKeeper实例时指定的，如果要设置特定的Watcher的话，可以调用另一个重载版本的exists(path, watcher)。以下几个带watch参数的API也都类似 。


1. getData(path, watch): 读取指定ZNode上的数据，并设置是否watch这个ZNode 。


1. setData(path, watch): 更新指定ZNode的数据，并设置是否Watch这个ZNode 。


1. getChildren(path, watch): 获取指定ZNode的所有子ZNode的名字，并设置是否Watch这个ZNode 。


1. multi(Iterable<Op> ops): 执行多个ZooKeeper操作，或者一个都不执行。


1. sync(path): 把所有在sync之前的更新操作都进行同步，达到每个请求都在半数以上的ZooKeeper Server上生效。path参数目前没有用 。


1. setAcl(path, acl): 设置指定ZNode的Acl（Access Control，访问控制）信息 。


1. getAcl(path): 获取指定ZNode的Acl信息 。

## 应用场景

本章主要内容摘录自参考文献[2]。

#### 命名服务(Name Service)

类似于DNS（域名系统，Domain Name System）。
分布式应用中，通常需要一套完备的命令机制，既能产生唯一的标识，又方便人识别和记忆。 我们知道，每个ZNode都可以由其路径唯一标识，路径本身也比较简洁直观，另外ZNode上还可以存储少量数据，这些都是实现统一的NameService的基础。 

#### 配置管理(Configuration Management)

在分布式系统中，常会遇到这样的场景: 某个Job的很多个实例在运行，它们在运行时大多数配置项是相同的，如果想要统一改某个配置，一个个实例去改，是比较低效，也是比较容易出错的方式。通过ZooKeeper可以很好的解决这样的问题，下面的基本的步骤：
将公共的配置内容放到ZooKeeper中某个ZNode上，比如/service/common-conf 。
所有的实例在启动时都会传入ZooKeeper集群的入口地址，并且在运行过程中监听 /service/common-conf这个结点。
如果集群管理员修改了了common-conf，所有的实例都会被通知到，根据收到的通知更新自己的配置，并继续监听 /service/common-conf 。

#### 组员管理(Group Membership)

在典型的Master-Slave结构的分布式系统中，Master需要作为“总管”来管理所有的Slave, 当有Slave加入，或者有Slave宕机，Master都需要感知到这个事情，然后作出对应的调整，以便不影响整个集群对外提供服务。以HBase为例，HMaster管理了所有的RegionServer，当有新的RegionServer加入的时候，HMaster需要分配一些Region到该RegionServer上去，让其提供服务；当有RegionServer宕机时，HMaster需要将该RegionServer之前服务的Region都重新分配到当前正在提供服务的其它RegionServer上，以便不影响客户端的正常访问。下面是这种场景下使用ZooKeeper的基本步骤：
Master在ZooKeeper上创建/service/slaves结点，并设置对该结点的Watcher 。
每个Slave在启动成功后，创建唯一标识自己的临时性(Ephemeral)结点/service/slaves/${slave_id}，并将自己地址(ip/port)等相关信息写入该结点 。
Master收到有新子结点加入的通知后，做相应的处理 。
如果有Slave宕机，由于它所对应的结点是临时性结点，在它的Session超时后，ZooKeeper会自动删除该结点 。
Master收到有子结点消失的通知，做相应的处理 。

#### 简单互斥锁(Simple Lock)

我们知识，在传统的应用程序中，线程、进程的同步，都可以通过操作系统提供的机制来完成。但是在分布式系统中，多个进程之间的同步，操作系统层面就无能为力了。这时候就需要像ZooKeeper这样的分布式的协调(Coordination)服务来协助完成同步，下面是用ZooKeeper实现简单的互斥锁的步骤，这个可以和线程间同步的mutex做类比来理解：
多个进程尝试去在指定的目录下去创建一个临时性(Ephemeral)结点 /locks/my_lock 。ZooKeeper能保证，只会有一个进程成功创建该结点，创建结点成功的进程就是抢到锁的进程，假设该进程为A 。
其它进程都对/locks/my_lock进行Watch 。
当A进程不再需要锁，可以显式删除/locks/my_lock释放锁；或者是A进程宕机后Session超时，ZooKeeper系统自动删除/locks/my_lock结点释放锁。
此时，其它进程就会收到ZooKeeper的通知，并尝试去创建/locks/my_lock抢锁，如此循环反复 。

#### 互斥锁(Simple Lock without Herd Effect)

上一节的例子中有一个问题，每次抢锁都会有大量的进程去竞争，会造成羊群效应(Herd Effect)，为了解决这个问题，我们可以通过下面的步骤来改进上述过程：
每个进程都在ZooKeeper上创建一个临时的顺序结点(Ephemeral Sequential) /locks/lock_${seq} 。${seq}最小的为当前的持锁者(${seq}是ZooKeeper生成的Sequenctial Number) 。其它进程都对只watch比它次小的进程对应的结点，比如2 watch 1, 3 watch 2, 以此类推 
当前持锁者释放锁后，比它次大的进程就会收到ZooKeeper的通知，它成为新的持锁者，如此循环反复 
这里需要补充一点，通常在分布式系统中用ZooKeeper来做Leader Election(选主)就是通过上面的机制来实现的，这里的持锁者就是当前的“主”。

#### 读写锁(Read/Write Lock)

我们知道，读写锁跟互斥锁相比不同的地方是，它分成了读和写两种模式，多个读可以并发执行，但写和读、写都互斥，不能同时执行行。利用ZooKeeper，在上面的基础上，稍做修改也可以实现传统的读写锁的语义，下面是基本的步骤:

每个进程都在ZooKeeper上创建一个临时的顺序结点(Ephemeral Sequential) /locks/lock_${seq} 
${seq}最小的一个或多个结点为当前的持锁者，多个是因为多个读可以并发 
需要写锁的进程，Watch比它次小的进程对应的结点 
需要读锁的进程，Watch比它小的最后一个写进程对应的结点 
当前结点释放锁后，所有Watch该结点的进程都会被通知到，他们成为新的持锁者，如此循环反复 

#### 分布式队列 

队列方面，简单地讲有两种，一种是常规的先进先出队列，另一种是要等到队列成员聚齐之后的才统一按序执行。对于第一种先进先出队列，和分布式锁服务中的控制时序场景基本原理一致，这里不再赘述。 
第二种队列其实是在FIFO队列的基础上作了一个增强。通常可以在 /queue 这个znode下预先建立一个/queue/num 节点，并且赋值为n（或者直接给/queue赋值n），表示队列大小，之后每次有队列成员加入后，就判断下是否已经到达队列大小，决定是否可以开始执行了。这种用法的典型场景是，分布式环境中，一个大任务Task A，需要在很多子任务完成（或条件就绪）情况下才能进行。这个时候，凡是其中一个子任务完成（就绪），那么就去 /taskList 下建立自己的临时时序节点（CreateMode.EPHEMERAL_SEQUENTIAL），当 /taskList 发现自己下面的子节点满足指定个数，就可以进行下一步按序进行处理了。

#### 屏障(Barrier)

在分布式系统中，屏障是这样一种语义: 客户端需要等待多个进程完成各自的任务，然后才能继续往前进行下一步。下用是用ZooKeeper来实现屏障的基本步骤：

Client在ZooKeeper上创建屏障结点/barrier/my_barrier，并启动执行各个任务的进程 
Client通过exist()来Watch /barrier/my_barrier结点 
每个任务进程在完成任务后，去检查是否达到指定的条件，如果没达到就啥也不做，如果达到了就把/barrier/my_barrier结点删除 
Client收到/barrier/my_barrier被删除的通知，屏障消失，继续下一步任务 

#### 双屏障(Double Barrier)

双屏障是这样一种语义: 它可以用来同步一个任务的开始和结束，当有足够多的进程进入屏障后，才开始执行任务；当所有的进程都执行完各自的任务后，屏障才撤销。下面是用ZooKeeper来实现双屏障的基本步骤：

进入屏障： 
Client Watch /barrier/ready结点, 通过判断该结点是否存在来决定是否启动任务 
每个任务进程进入屏障时创建一个临时结点/barrier/process/${process_id}，然后检查进入屏障的结点数是否达到指定的值，如果达到了指定的值，就创建一个/barrier/ready结点，否则继续等待 
Client收到/barrier/ready创建的通知，就启动任务执行过程 
离开屏障： 
Client Watch /barrier/process，如果其没有子结点，就可以认为任务执行结束，可以离开屏障 
每个任务进程执行任务结束后，都需要删除自己对应的结点/barrier/process/${process_id} 

#### 集群管理

集群机器监控：这通常用于那种对集群中机器状态，机器在线率有较高要求的场景，能够快速对集群中机器变化作出响应。这样的场景中，往往有一个监控系统，实时检测集群机器是否存活。过去的做法通常是：监控系统通过某种手段（比如ping）定时检测每个机器，或者每个机器自己定时向监控系统汇报“我还活着”。 这种做法可行，但是存在两个比较明显的问题： 
集群中机器有变动的时候，牵连修改的东西比较多。 
有一定的延时。 
利用ZooKeeper有两个特性，就可以实时另一种集群机器存活性监控系统：

客户端在节点 x 上注册一个Watcher，那么如果 x?的子节点变化了，会通知该客户端。 
创建EPHEMERAL类型的节点，一旦客户端和服务器的会话结束或过期，那么该节点就会消失。 
例如，监控系统在 /clusterServers 节点上注册一个Watcher，以后每动态加机器，那么就往 /clusterServers 下创建一个 EPHEMERAL类型的节点：/clusterServers/{hostname}. 这样，监控系统就能够实时知道机器的增减情况，至于后续处理就是监控系统的业务了。

#### Master选举

Master选举则是zookeeper中最为经典的应用场景了。 
在分布式环境中，相同的业务应用分布在不同的机器上，有些业务逻辑（例如一些耗时的计算，网络I/O处理），往往只需要让整个集群中的某一台机器进行执行，其余机器可以共享这个结果，这样可以大大减少重复劳动，提高性能，于是这个master选举便是这种场景下的碰到的主要问题。

利用ZooKeeper的强一致性，能够保证在分布式高并发情况下节点创建的全局唯一性，即：同时有多个客户端请求创建 /currentMaster 节点，最终一定只有一个客户端请求能够创建成功。利用这个特性，就能很轻易的在分布式环境中进行集群选取了。

另外，这种场景演化一下，就是动态Master选举。这就要用到?EPHEMERAL_SEQUENTIAL类型节点的特性了。

上文中提到，所有客户端创建请求，最终只有一个能够创建成功。在这里稍微变化下，就是允许所有请求都能够创建成功，但是得有个创建顺序，于是所有的请求最终在ZK上创建结果的一种可能情况是这样： /currentMaster/{sessionId}-1 ,?/currentMaster/{sessionId}-2 ,?/currentMaster/{sessionId}-3 ….. 每次选取序列号最小的那个机器作为Master，如果这个机器挂了，由于他创建的节点会马上小时，那么之后最小的那个机器就是Master了。

在搜索系统中，如果集群中每个机器都生成一份全量索引，不仅耗时，而且不能保证彼此之间索引数据一致。因此让集群中的Master来进行全量索引的生成，然后同步到集群中其它机器。另外，Master选举的容灾措施是，可以随时进行手动指定master，就是说应用在zk在无法获取master信息时，可以通过比如http方式，向一个地方获取master。 
在Hbase中，也是使用ZooKeeper来实现动态HMaster的选举。在Hbase实现中，会在ZK上存储一些ROOT表的地址和HMaster的地址，HRegionServer也会把自己以临时节点（Ephemeral）的方式注册到Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的存活状态，同时，一旦HMaster出现问题，会重新选举出一个HMaster来运行，从而避免了HMaster的单点问题 。

## 安装配置

ZooKeeper的安装模式分为三种，分别为：单机模式（stand-alone）、集群模式和集群伪分布模式。

#### 单机模式

tickTime=2000 dataDir=/Users/apple/zookeeper/data dataLogDir=/Users/apple/zookeeper/logs clientPort=4180下载zookeeper的安装包之后, 解压到合适目录. 进入zookeeper目录下的conf子目录, 创建zoo.cfg:  



参数说明: tickTime: zookeeper中使用的基本时间单位, 毫秒值. dataDir: 数据目录. 可以是任意目录. dataLogDir: log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和dataDir相同的设置. clientPort: 监听client连接的端口号.

#### 伪集群模式

所谓伪集群, 是指在单台机器中启动多个zookeeper进程, 并组成一个集群. 以启动3个zookeeper进程为例.
将zookeeper的目录拷贝2份。zookeeper0/conf/zoo.cfg文件为:



``` 
tickTime=2000

initLimit=5

syncLimit=2

dataDir=/Users/apple/zookeeper0/data

dataLogDir=/Users/apple/zookeeper0/logs

clientPort=4180

server.0=127.0.0.1:8880:7770

server.1=127.0.0.1:8881:7771

server.2=127.0.0.1:8882:7772
```



新增了几个参数, 其含义如下:
1 initLimit: zookeeper集群中的包含多台server, 其中一台为leader, 集群中其余的server为follower. initLimit参数配置初始化连接时, follower和leader之间的最长心跳时间. 此时该参数设置为5, 说明时间限制为5倍tickTime, 即5*2000=10000ms=10s.
2 syncLimit: 该参数配置leader和follower之间发送消息, 请求和应答的最大时间长度. 此时该参数设置为2, 说明时间限制为2倍tickTime, 即4000ms.
3 server.X=A:B:C 其中X是一个数字, 表示这是第几号server. A是该server所在的IP地址. B配置该server和集群中的leader交换消息所使用的端口. C配置选举leader时所使用的端口. 由于配置的是伪集群模式, 所以各个server的B, C参数必须不同.
参照zookeeper0/conf/zoo.cfg, 配置zookeeper1/conf/zoo.cfg, 和zookeeper2/conf/zoo.cfg文件. 只需更改dataDir, dataLogDir, clientPort参数即可.
在之前设置的dataDir中新建myid文件, 写入一个数字, 该数字表示这是第几号server. 该数字必须和zoo.cfg文件中的server.X中的X一一对应.
/Users/apple/zookeeper0/data/myid文件中写入0, /Users/apple/zookeeper1/data/myid文件中写入1, /Users/apple/zookeeper2/data/myid文件中写入2.
分别进入/Users/apple/zookeeper0/bin, /Users/apple/zookeeper1/bin, /Users/apple/zookeeper2/bin三个目录, 启动server.

#### 集群模式

集群模式的配置和伪集群基本一致.
由于集群模式下, 各server部署在不同的机器上, 因此各server的conf/zoo.cfg文件可以完全一样.下面是一个示例:
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/home/zookeeper/data
dataLogDir=/home/zookeeper/logs
clientPort=4180
server.43=10.1.39.43:2888:3888
server.47=10.1.39.47:2888:3888
server.48=10.1.39.48:2888:3888









示例中部署了3台zookeeper server, 分别部署在10.1.39.43, 10.1.39.47, 10.1.39.48上. 需要注意的是, 各server的dataDir目录下的myid文件中的数字必须不同，10.1.39.43 server的myid为43, 10.1.39.47 server的myid为47, 10.1.39.48 server的myid为48.



## 性能测试

本章主要内容摘录自参考文献[3]。

#### 硬件配置

在3台机器上分别部署一个zookeeper，版本为3.4.3，机器配置：
Intel(R) Xeon(R) CPU E5-2430 0 @ 2.20GHz  
16G  
java version "1.6.0_32"  
Java(TM) SE Runtime Environment (build 1.6.0_32-b05)  
OpenJDK (Taobao) 64-Bit Server VM (build 20.0-b12-internal, mixed mode)  
JVM堆大小：1/4 RAM

#### 节点数对读写性能的影响

测量最大10万个节点，度量1秒内操作数（ops）：

结论：对单个节点的操作并不会因为zookeeper中节点的总数而受到影响。

#### 节点数据大小对读写性能的影响

本身单个节点数据越大，对网络方面的吞吐就会造成影响，所以其数据越大读写性能越低也在预料之中。写数据会在zookeeper集群内进行同步，所以其速度整体会比读数据更慢。该实验需要把超时时间进行一定上调，同时我也把JVM最大堆大小调整到8G。测试结果很明显，节点数据大小会严重影响zookeeper效率。

#### watch对读写性能的影响

通过创建多个客户端来模拟单个节点上的多个watch。这也更符合实际应用。同时对节点的写也是在另一个独立的客户端中，这样可以避免zookeeper client的实现对测试带来的干扰。
每一次完整的测试，首先是对每个节点添加节点数据的watch，然后在另一个客户端中对这些节点进行数据改写，收集这些改写操作的耗时，以确定添加的watch对这些写操作带来了多大的影响。
图中，0 watch表示没有对节点添加watch；1 watch表示有一个客户端对每个节点进行了watch；3 watch表示有其他3个客户端对每个节点进行了watch；依次类推。
可见，watch对写操作还是有较大影响的，毕竟需要进行网络传输。同样，这里也显示出整个zookeeper的watch数量同节点数量一样对整体性能没有影响。

## 注意事项

1. ZNode数据存储大小默认为1MB，可以通过配置参数进行调节；


1. 对某一个结点进行写操作，其性能受该结点上的watch（即有多少客户端监听该结点）影响，但是与服务器上其他结点上的watch数量无关。

