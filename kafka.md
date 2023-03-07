kafka是一种消息队列

消息队列的好处:1.解耦合   2.异步处理    3.流量的削峰

![1676945281717](C:\Users\2541843624\AppData\Roaming\Typora\typora-user-images\1676945281717.png)

kafka的消费模式有两种:

 		一对一:这种消费模式,消息存储在queue中,在数据被消费以后,数据会被删除

​		一对多:也叫做订阅者模式,数据存储在topic中,消费者消费数据之后不会被删除,默认保留一段时间

partition:分区,为了实现扩展性,每个partition都是一个有序的消息队列,但不能保证全局有序

Producer生产的数据会被不断的追加到该log文件的末端，且每条数据都有自己的offset，consumer组中的每个consumer，都会实时记录自己消费到了哪个offset，以便出错恢复的时候，可以从上次的位置继续消费。流程：Producer => Topic（Log with offset）=> Consumer.

分区的原因

​			方便在集群中扩展：每个partition通过调整以适应它所在的机器，而一个Topic又可以有多个partition组成，因此整个集群可以适应适合的数据
可以提高并发：以Partition为单位进行读写。类似于多路。
分区的原则

​			指明partition（这里的指明是指第几个分区）的情况下，直接将指明的值作为partition的值
没有指明partition的情况下，但是存在值key，此时将key的hash值与topic的partition总数进行取余得到partition值
值与partition均无的情况下，第一次调用时随机生成一个整数，后面每次调用在这个整数上自增，将这个值与topic可用的partition总数取余得到partition值，即round-robin算法。
Kafka消费者分区分配策略:round-robin循环 ,range



kafka的高效读写: 顺序写磁盘,零复制技术  







Kafka 中的 ISR(InSyncRepli)、 OSR(OutSyncRepli)、 AR(AllRepli)代表什么？

ISR：速率和leader相差低于10s的follower的集合

OSR：速率和leader相差大于10s的follwer

AR：所有分区的follower

Kafka 中的 HW、 LEO 等分别代表什么？

HW：High Water高水位，根据同一分区中最低的LEO决定（Log End Offset）

LEO：每个分区最大的Offset

Kafka 中是怎么体现消息顺序性的？

在每个分区内，每条消息都有offset，所以消息在同一分区内有序，无法做到全局有序性

Kafka 中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？

分区器Partitioner用来对分区进行处理的，即消息发送到哪一个分区的问题。序列化器，这个是对数据进行序列化和反序列化的工具。拦截器，即对于消息发送进行一个提前处理和收尾处理的类Interceptor，处理顺利首先通过拦截器=>序列化器=>分区器

Kafka 生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？

使用两个线程：main和sender 线程，main线程会一次经过拦截器、序列化器、分区器将数据发送到RecoreAccumulator线程共享变量，再由sender线程从共享变量中拉取数据发送到kafka broker

batch.size达到此规模消息才发送，linger.ms未达到规模，等待当前时长就发送数据。

消费组中的消费者个数如果超过 topic 的分区，那么就会有消费者消费不到数据”这句 话是否正确？

这句话是对的，超过分区个数的消费者不会在接收数据，主要原因是一个分区的消息只能够被一个消费者组中的一个消费者消费。

消费者提交消费位移时提交的是当前消费到的最新消息的 offset 还是 offset+1？

生产者发送数据的offset是从0开始的，消费者消费的数据的offset是从1开始，故最新消息是offset+1

有哪些情形会造成重复消费？

先消费后提交offset，如果消费完宕机了，则会造成重复消费

那些情景会造成消息漏消费？

先提交offset，还没消费就宕机了，则会造成漏消费

当你使用 kafka-topics.sh 创建（删除）了一个 topic 之后， Kafka 背后会执行什么逻辑？

会在 zookeeper 中的/brokers/topics 节点下创建一个新的 topic 节点，如：/brokers/topics/first
触发 Controller 的监听程序
kafka Controller 负责 topic 的创建工作，并更新 metadata cache

topic 的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？

可以增加，修改分区个数--alter可以修改分区个数

topic 的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？

不可以减少，减少了分区之后，之前的分区中的数据不好处理

Kafka 有内部的 topic 吗？如果有是什么？有什么所用？

有，__consumer_offsets主要用来在0.9版本以后保存消费者消费的offset

Kafka 分区分配的概念？

Kafka分区对于Kafka集群来说，分区可以做到负载均衡，对于消费者来说分区可以提高并发度，提高读取效率

简述 Kafka 的日志目录结构？

每一个分区对应着一个文件夹，命名为topic-0/topic-1…，每个文件夹内有.index和.log文件。

如果我指定了一个 offset， Kafka Controller 怎么查找到对应的消息？

offset表示当前消息的编号，首先可以通过二分法定位当前消息属于哪个.index文件中，随后采用seek定位的方法查找到当前offset在.index中的位置，此时可以拿到初始的偏移量。通过初始的偏移量再通过seek定位到.log中的消息即可找到。

聊一聊 Kafka Controller 的作用？

Kafka集群中有一个broker会被选举为Controller，负责管理集群broker的上下线、所有topic的分区副本分配和leader的选举等工作。Controller的工作管理是依赖于zookeeper的。

Kafka 中有那些地方需要选举？这些地方的选举策略又有哪些？

在ISR中需要选举出Leader，选择策略为先到先得。在分区中需要选举，需要选举出Leader和follower。

失效副本是指什么？有那些应对措施？

失效副本为速率比leader相差大于10s的follower，ISR会将这些失效的follower踢出，等速率接近leader的10s内，会重新加入ISR

Kafka 的哪些设计让它有如此高的性能？

Kafka天生的分布式架构
对log文件进行了分segment，并对segment建立了索引
对于单节点使用了顺序读写，顺序读写是指的文件的顺序追加，减少了磁盘寻址的开销，相比随机写速度提升很多
使用了零拷贝技术，不需要切换到用户态，在内核态即可完成读写操作，且数据的拷贝次数也更少。
