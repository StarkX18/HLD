Problem-2

Async tasks / low latency

=> suppose a messaging system: when message arrives : save, notify (-> in-app / push or email based on activity of user), do some analytics, fraud/safety filtering etc

=> if each service takes some time, say 5s -> success response will be sent after 5s to each message!

=> all these tasks are not important wrt user, only save is important immediately

=> save is hence performed synchronously, but the other tasks are async

- this is to ensure low latency and success response is sent at the earliest

But sometimes even though they are carried on later on, we might want to ENSURE the success of these tasks, without fail!

-> persistent queues! stored on disk/replicas

other use cases for persistent queues?
- shock absorbers for spikes in requests - send success response as soon as enqueued or tell them to check later on

Persistent Queues work on a pub-sub model

- message queue/ message broker / persistent queue

- clients are machines that are generating data and pushing events into the queue

- at the other end, we have consumers, which take data and process it

Kafka - one such MQ

- internally uses ZK - for replication and disk write
- each kafka queue is divided into multiple topics

- eg for flipkart: buy a product or text the supplier

- when we buy a product: notify, invoice, email, analytics
- when message to supplier: email, call after delay, analytics for reputatin of supplier


- if EVERY event is pushed into same queue, too much noise, and latency per consumer

- any event relevant to the topic is pushed into this topic/subqueue

Producers and consumers can work at different rates

- it is guaranteed that kafka will store the events on disk, hence consumers can work at their own rate

- also one event maybe consumed by multiple consumers

- any data produced is by default saved for a week during which any consumer may consume it

- the data is deleted after the retention period though

Cant the same event be consumed multiple times then? we will see later

Every topic is further sub divided into several partitions

-> since even each topic might not be consumable by the consumers inside the retention period

-> data can either go to random or specific partition

-> partition is a way to shard the data

-> now, each consumer is tied to one partition
-> but each partition may have multiple consumers

-> producer may transmit to multiple partitions

-> no consumer can be tied to multiple patition - ensured by ZK

-> num_parition <= consumers for the topic

By default, events are assigned to partition in round robin

- now suppose uber like system
- driver locations are being pinged by drivers
- one consumer is charting the path on a map

- RR will be a problem here, but not in all use cases like emails etc

But kafka also allows you to supply key

- partition number = hash(key) % nnum_parition

- hence same driver is assigned the same partition always

- if we need ordered processing of messages, then we can use key while publishing?

- just key will not be sufficient - since consumer should consume in which inserted into Q

- however what is imp is that it is written to a single partition

If a topic is huge

=> 1 consumer - long time to consume

consumer groups - managed by consumer

consumer tells kafka which topic it is listening to, and to which group it belongs

then kafka handles which partition is allotted to which consumer

consumer doesnt decide which partition it will listen to

what happens when a single machine/broker is dead in kafka?

replication, as always

each partition is replicated multiple times and replication factor is defined for partition

what problem happens with distributed architecture?
- which machine does the external service talk to?

- talk to any machine - kafka automatically redirects

whenever we want multiple consumers to read from the same topic - do they need to be in same consumer group?
prolly not - check


- inside a particular consumer group, one message from a topic will only be consumed by one consumer

How is Fifo maintained? offset -> increases monotonically

how consumption happen in group to avoid duplicate subscription


Two Phase Commit https://www.youtube.com/watch?v=-_rdWB9hN1c 

https://youtu.be/eltn4x788UM

https://dattell.com/data-architecture-blog/understanding-kafka-consumer-offset/

https://docs.google.com/document/d/1jRRtAl6Iqt-_x4Gzx4AcK4vay4ftTvqsWwe31ZmhnKk/edit

https://docs.google.com/document/d/1FCi-xxi0HgR7he7Dsat2hsgiMFIiffSrLCecB6-BfT4/edit

https://apache.googlesource.com/zookeeper/+/3d2e0d91fb2f266b32da889da53aa9a0e59e94b2/src/java/main/org/apache/zookeeper/server/quorum/FastLeaderElection.java

In the uber example, we’re pushing driver location to a specific partition using hash function. But on the other end consumer is assigned partition by Kafka. How do we make sure a particular driver’s location is sent to the correct consumer?

replication of kapfa is not clear?

can you give overviw on kraft based qourum?

How do we decide number of partitions while creating topic... Lets say I have 1000 writes/sec to topic and my consumer instance takes 100ms for processing a single request..


Is the messages from the queue are flushed over after all consumers have consumed it or it is permanently stored ?

So whatever slave don't have updated data and the time they got to know master is changed by the same time whatever master(this is upstaed master) we make that down so he will also participitae in election without complete data replica and think that salve assign as a master then this master don't have updated data right. how to deal woth this satuation.s 

How in case of kafka we dont loose message when zookeeper switches master ?

how the slaves know which is master is it thourg zoo keeper only or any other mechanism?

 how many app server can write the data