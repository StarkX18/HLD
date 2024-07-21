Tarun malhotra: google :o
- https://www.linkedin.com/in/tarun-malhotra-72973aa6/

Iterinary of a request

1. client -> dns: got IP
2. client -> gateway box / load balancer -> the first box that interacts with outside world?
also acts as a load balancer,

plus also a bunch of things like - https to http etc
is that a regular norm followed by most companies where ssl terminates at gateway?

routing
Static cache
what about WAF?
URL rewriting, rate limiting

3. bunch of app-servers -> here need the "elasticity" / flexibility to scale up and down

4. storage layer: databases etc - source of truth...

what is serverless?

CAP theorem...
- sharding -> partitioning -> CAP

sharding: mutually exclusive and exhaustive data!!!

example:
1. fb - 3B net, 1B MAU, 0.5B DAU

A. user-db: user_id, name, contact details(phone, email, address etc)
B. user-friend: user_id/friend_id

single machine wont support : 100TB data already

sharding strategy can be random, time-based, geographical etc - depending on use case!!!

I have one doubt:
For User and UserFrnd, we can divide them in 3 ways:
Put User in 1 m/c and Frnd in another m/c 2. Only User table, keep 1B records in 1 m/c and 2B records in another m/c. 3. For User table, Keep all 3B records but with some columns in 1 /mc and remaining columns in another m/c.
Which one is true?

do we want to store driends along with user?

but if a friend belongs to 2 friends then don't we have duplicacy? If user_id as shrading key?

yes - joins easier that way prolly but dont know the full answer!

if we keep shard a user's friend data also in same shard then are we not going to have duplicate data in another shard where shard a user friends are there?

sharding and replication are kind of opposite - almost opposite properties in terms of data duplicacy, and mutual exclusiveness

however they can and will exist simultaneously !

storage is a lot more stable, static and durable.
compute is erratic, dynamic and ephemeral!

but we might not know the future right, so how is it predicted about the number of shards that would be required?

what is implication on bandwidth when you replicate whole data?

concept of hot shards and cold shards

concept of intra shard requests and inter shard requests!

intra is optimal - inter is dreadful.
we design sharding so as to maximise intra shard.

Pragy's example on fb feeed

CAP theorem:

In distributed systems - you can only choose 2 out of these 3 properties!

C - consistency: every read should return the latest write
A - availability: 
P - Partition tolerance

Its like heisenberg's uncertainity principle

distributed systems? not a single machine but multiple machines do the same job.

consistency: every read - no matter from wherever - should return the LATEST write!

every machine has the same latest view of the truth

left a few mins back - start from hardeep nikhil example:

reminder service with fancy number, 99991111 - takes off!

-> write and read request - tell to remind, and actually remind!

-> hires nikhil: some hardware - forwards request between hardeep and nikhil

-> but both will not save each other's diaries - so might lead to conflicts!

-------------------------------------------------------------
explaining consistency

ways to make it consistent - specific case:

# write requests - respond success only after ensuring that the other person has also written the record in his diary.
- basically replication

# cons:
- slower write requests
- if machine is down temporarily.

- ie if fo

given the prev protocol: if one of the machine is down: the whole system is down!

availability: 
- you are available to register requests
- whenever queried you should be able to respond back without any errors!
- availability doesn't guarantee CORRECT responses - it only guarantees the system's presence to serve repsonses!

consistency implied that all the machinesa will have the same knowledge and hence all queries will have correct responses!

availability only implied server's presence and responses - maybe not all machines are up, or have the same knowledge even!

example:
YT - view count on youtube

see a video, see after 10 mins. but maybe view count doesnt change!

maybe view count isnt always correct but tis always there!
similarly for hotstar!


zerodha : stock prices: consistency is FAR FAR more important than availability!

ATM/zerodha/banking - service unavailable > wrong data!

right now the system is inconsistent - but available.

but now - when you prefer consistency - you loose out on availability because

- suppose nikhil is out temporarily (unavailable) - responses are fail!

so when consistency is preferred - system is more prone to unavailability

solving prev problem:
updated protocol.
- nikhil is on leave - unavailable
- hardeep now notices he can atleast write down the entries on his diary
- when nikhil is back - will sync our entries!

- message queue, db etc - then cron job or healthcheck as triggers to push or pull data

now the system is consistent before the capsized server startts serving requests!

they are also available - atleast at some point of time!

now comes partition tolerance.
partition here = network partition.
- network partition means network is broken temporarily between any two machines temporarily!

this is NOT data partitioning referemced in sharding.

nikhil asks for a raise - infighting breaks out and they stop talking - network partitioning

network is always unreliable!

what happens if - DURING the sync of nikhil and hardeep -> network partition happens?

sync is interrupted!

partition tolerance implies the system is ok with network breaks.

ONLY if the third always holds true -> which isnt really the case in real life -> network is ALWAYS unreliable!

in a distributed system -> only if you dont have partitions will you be able to be available and consisyent always!

thus - in distributed systems - partition tolerance is ALWAYS required.
So they are always CP, or AP.

why was this law framed this way then?
why not cp/ ap law?
For a single maching however - can be both CA at same time!

now onto the actual system:

option 1: both write separately - no notes sharing

system is NOT consistent. available. and partition tolerant. thus AP.

option 2: both write down - response is success iff both have written down.

system is consistent. not available if any one goes down. partition tolerant. - why because you can allow partitions and respond.

option 3: when any is down - write on remaining machine - sync up before it comes back up.

system is available. partition tolerant. not consistent

Can partition on single machine (like multiple DB partitions on same machine) bring CAP back? Even such IO between DB partition is also Partition?

If you want to save on consistency(reads ALWAYS give latest write) and availability(meaning - if a machine is live to take it - take it)

- you can NOT be tolerant to partitions ie network partition!


conlusion again : partition is a given - hence SHOULD be partition tolerance in any real world system

- thus CP or AP only

PACELC theorem - 

PAC: whenever partition happens - prefer availability or consistency

E: else ie when partition is NOT happening

LC: choose between latency and consistency

latency: time taken by a system to process a request


whenever partition happens : A or C
whenever its not happening: L or C
either low latency or high consistency

there's a latency in IRCTC/Airports - since they want super consistency so no two people book with same seat - 

tAking extra pain yo make the content harder so that it is done right! -> because we want to be super consistent - hence latency increases

why:
highly consistency: all db write = more db writes
db reads are same

low latency: one db writes = less db writes
db reads are same

dont refresh !! your transaction is in progress

Confused:
When partition is not happening, doesn't it mean single system? Latency in terms of write on multiple server, means this is distributed in turns partition is there already?

then it means IRCTC has one one system/server(single box)

booking systems are highly consistent.. media system dont need strict consistency so they use eventual consistency

In case of no partition it can be either having distributed cache / sharding / consistent hashing it would increase latency ...

replication:
very opposite from sharding 

A particular architecture
- called master-slave
- what else? suggestions?
   - orchestrator and worker
   - single leader
   - leader follower
   - active active?

master - for writes
slaves - for reads!

one master - many slaves

coz reads>>writes!

gmail - not that much of a gap
social media - read heavy
Are likes and comments not treated as "writes" ?

wrote to one: write to others offline

-> low latency - less time
-> low consistency! - no accuracy - eventuaal consistency

wrote on master - write to all slaves
- high consistency
- high latency!

- so CAP theorem is culprit behind IRCTC high latency !!

Other ways of replication.

-  dont wait for all slaves, but a small portion only -> then respond success;

- problem with eventual consistency - what if something physically bad happens to master?

- linkedin/insta feed: kinda same - as discussed before!

- fb likes/insta likes -> only master -> occassionally get lost/dont register?

- financial / bank: consistent, latency ok, availability ok
- fb posts - availability, not consistency
- low latency, not high consistency

- chat systems: consistent!
- multiplayer games: availability>>consistency, plus low latency!!

- whatsapp: lower avaialbility, high consistency

the architecture depends on business use case, not technical :P

how does caching operate with master-slave?

good question!

whatsapp use case:

single tick shown, not double tick - means prolly high latency on whatsapp too!


DIY: work with iMessage!

in distributed cache like redis, does follow master slave or master/leader less ?

are caches also replicated at times?
Sawan Kumar
To: Everyone
12:06 AM
strategies for dealing with heavy writes to a DB

master election protocol -etc


how replication happens in the slaves if new writes are performed on the master? can we discuss more scenarios when slaves can go down or master can go down?

Can you throw some light on CAP vs ACID in case of databases? 

why network is called unreliable? For eg, lets say there is a subnet which has 10 machines, inside the subnet how can the network be unreliable?

which protocol used for communication between replicas ? please discuss master slave vs peer to peer and other replication strategy

how we will deal with HTTP request consider a case request recived first serivice is down where we are making copy till that we will not response so previous we are reponsinding in 5 sec now how we will  deal with this case

during replication is the source node brought down or writes keep on happening on it?

how do we choose the value of X slaves to write in scenario 3? how it is helpful?

what is more important for trading platforms in terms of CAP and PACELC?

there will be multiple tables present where user_id might be the primary key, so for a particular user_id, all data for each table will be present in a single shard? 

suppose from client side, I want to access 2 user in 1 request, user1 is in shard A, user2 is in Shard B, how retrieval happens? What happens to request, how does it get processed?

does mysql or any db natively support sharding by default? if not, how to configure?

if there is a table that does not have directly user_id, how does the data in that split if the sharding is based on user_id?

So in case of DB down - migration of data will happen right? so for EACH row - we will calculate shard number and send it over there? 
isnt this very slow?

which system costs more to maintain ? Does costs has part to play in choosing a design ?

Can you explain user and friend's data how are they in same Shard? So in that case friend of friend should also have data in same shard?
