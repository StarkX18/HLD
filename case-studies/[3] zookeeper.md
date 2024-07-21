hi one doubt pragy can we read ops by Btree and write ops by lsm tree is that possible?

no - if you are writing data into some DS -> the data is stored in the structure of the same DS right?

One thing -> simultaneously store in form of B Tree -> but no seperate speedup -> SS tables are already sorted and binary search with sparse index is also present

Our table is prolly better than standard BTree!

Agenda:

1. Problem Statement
2. Zookeeper and Kafka as solutions!

Problem 1: Consistent State Tracking

- If we want to see that the reads stay true: we use master slave architecture.
- now replicaation is usually applied to db: write got to master, reads go to slaves

- for this to happen - all the seperate app servers mUST know who is the maste at ANY and ALL points of time!

- cases like master dies, master is down temporarily etc - must update who the DB is in ALL the app servers is!

Problem: keep track of master ACROSS app servers!

- suggestions: 

1. (mine) maintain and keep refreshing using healtyhcheck - who master is - store in redis!

2. Gossip protocol?

3. common coordinator / orchestrator?
- not an orchstrator - it hasa much more complicated job!


So - a seperate machine that keeps track of who the master is...
and who the slavces are...

- for each request: it either responds to app server or forwards request directly to required master

- this means: 
- extra hop for each request.
- single point of failure machine!

to remove single point of failure:

- introduce multiple machines
- P1: but now all machines must agree on the same master!
- P2: and extra hop still there 

Here's where ZooKeeper comes in...

ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.

Leaving evertyhing aside: zookeeper stores configuration data in a strongly consistent manner!

Data storage behaves like a file system!

Also - each file is called a node -> in conflict with nomenclature for distributed systems -> where each machine is a node

will it not be overhead on network?
data means configurations? yes, mostly ephemeral nodes store data

Now, explaining zookeeper...

Un ZK - nodes != machine
ZK node = file

Types of ZK Nodes -
- persistent nodes: data persists until explicitly deleted
- ephemeral nodes: deleted conditionally: persists only as long as owner is live / available -
owner = machine in this case

- checked using heartbeat system at fixed interval 
-> if you dont get heartbeat - delete it :)

- how many attempts before delete the data in ZKnode ?

- Is there any chance other app server/db server write to same file?

This can solve our master-slave conundrum ->

everytime a master is elected. 
- our healthcheck fails - ephemeral node is deleted
- new master writes to this node

- node?   /master - stores IP of new master - then clears its contents

Master Election: how is the master elected?

- for now lets asssume ZK is a single machine - every machine can read through who the master is, using ZK

- for re-election: every slave tries to write its own IP inside ZK
- while writing - a lock is acquired - only the fastest fingers first slave will acquire the lock!
- once elected, this gives a heartbeat to ZK 

- now this slave might not be having the latest data * lets not worry about that right now...

- as soon as data written -> slave becomes new master!

will it not be overhead on network? YES

Problem of multiple hops

notification sent by zk is time based or event based? event based.

but data written of master 1 will also be lost.

What happens if zookeeper fails to notify?
how d oes zookeeper communicate with slaves ? tcp or rpc 

so the zookeeper comes in between the master and the slaves too?

Multiple hops issue... and how is election intiated?
- appserver -> ZK -> master
- instead of asking from ZK who the master is - ZK tells you who the master is...
- ZK has a subscription service: every machine can subscribe to a particular node on the ZK...
- so each server subscribes to the /master ... file 

- and know when any changes are made to this file 
- they also initiate write requests of their own IP to the ZK...

what happens to write requests during transition of master from old to new master

- also DURING the time that this whole thing is going on ie UNTIL new master is re-elected - ALL write requests are rejected

what if partition exists during this whole re-election between ZK and other nodes?


the moment app server detects that it is no longer connected to zookeeper

=> or it gets a notificastion from ZK, saying master is null

=> it must STOP serving any write requests.

the moment ZK does master re - election, it sends a notification to app server.

If this notification fails to deliver or something, the app server will still keep writing to prev master.

Similarly partition between other machines becomes even more problematic.

So during the re-election time - app servers and zookeeper and slave machines must, MUST have 2 way connections

not 1 way connection

hence the extra hop is solved by 2-way subscription!

In my personal experience, systems like kafka which uses zookeeper doesnt have ever loss of events in production.. not sure of probablity of master going down

subscription is triggered when there is a delta at the zknode or it checks frequently?

how frequent does master re-election happens ?
happens once in a MONTH, or WEEK

hence extra hopping = solved, because is costly but rare

what happens when one of the slave is not synced with the new master and it still tries to replicate from the old master? 

this app server works like client so why my client wait for my system is in maintain statate . we need to build somethis so app server don't bother about whatt happend with master and slave we will manage internalli

Does ZooKeeper also support persistent nodes?

can you give example where to use epermal and persistent nodes? 

What happenes if network segmentation happens between app server and zookeeper but app server can still connect to old master?

please provide a real life example of persistent zk nodes amd ephermal zk nodes ?

service discovery is ephemeral

internal architecture of zookeeper

zookeeper architecture: 

1) single point of failure? ZK isnt a single machine - we always do replication for this problem.

=> ZK is internally as well, leader-follower architecture

when master goes down, slaves write all this data to leader!

what sort of consistency is ZK looking for? Strong or eventual etc?

Strong consistency + leader follower = write to leader + write to a majority of slaves or all slaves!

number of machines = odd number = leader + even number of machines

2N+1 machines = leader + N+1 machines
else write fails!

leader commits this data and success response else rollback.

if fail, might succeed when the next slave tries to write


now 2 phase commit => first write and persist on leader upon write request

this data is however in non committed phase => using status flag + disk persistence

when it writes to disk, first leader, then followers, in NON COMMITTED phase

once a slave has written on disk, this follower has also not committed. But it sends acknowledgement to leader.

If sufficient slaves get written with this data, leader commits.

Similarly for followers, who will then commit on their respective disks.

this is the overview, not the full view

because here as well, partition can happen at any instance!

who will commit either follower or leader

DIY: 2 phase commit reading in detail

in a write heavy system, doesn't it become quite heavy or increases the load on traffic?


what about data in followers? sorry, I missed. is the change committed on follower?

hence our write succeeds only if n+1 followers are written onto.

why do we stick to N+1 reads?

because during read, we will only consider read, when the read from multiple servers is done and attain a quorum.

but we could have read from all not updated slaves? is that possible - no without getting half

split brain scenario

so if some followers go down, dynamically machine count can still become erqual to @N - NO

we keep the count to N+1 reads still!

so ZK is CP ?
not our entire system, but ZK internally is CP

how zookeeper's leader election and re-election happens?

-> when leader dies, re-elect the leader.
-> when a ZK machine goes down, its not a big problem. why?

-> when leader down, writes are blocked but reads aren't! 
-> db writes and reads, still happenning
-> any app-server can still read.
-> 

db servers are taking help of zoo keeper for reelection , but internally how do zk nodes coordinate with each other for reelection?

but, you said zkp publishes to app server which is the master

but here could be chance that at the same time zk'leader down as well as master dies?

its rare, but its down for a short period of time

but machines dont read from ZK right? it is based on subscription based mechanism. So, when leader is down, both writes and reads must stop right?

master re-election algorithm

but machines dont read from ZK right? it is based on subscription based mechanism. So, when leader is down, both writes and reads must stop right?