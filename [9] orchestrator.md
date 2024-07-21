Agenda: 
- Designing the orchestrator
- multi-master

Benefit of nosql - sharding. How exactly?
How is replication handled?

Problem statement: Design a "data" orchestrator / name node(hadoop) / job trapper / hbase-master - which has the following responsibilities...

1. automatically add or remove machines as needed
    - create new shards
    - assign shard to new servers

2. ensure equal load distribution at all times - consistent hashing, nothing fancy

3. maintaining replication level

Is Kubernetes also Orchestrator ?
Given config:
1. replication level = 3
2. sharding key = user_id
3. 10 servers

Problems 
- when exactly do we need to add new servers?

- something like load factor in hashing? if it has reached a certain limit of users/resources used?

- if we migrate users at peak load - it will only INCREASE traffic and latency
- hence being REACTIVE is not the right approach

- we will use consistent hashing we would be distributing all the load from 10 to 12 to equally distribute it?


If we let sharding happen at any timee : reacting to load -> create shard during peak load ->
move data from 1 machine to another -> increased traffic and load.

-> so lets not REACT to load? analyse historic data and work according to that?

---------------------------------------------------
if we have three copies of all data we can shard on one copy and do it passively on the other two copies !
---------------------------------------------------
replication levels: 

- lets not react to load: wen we have extra capacity -> assign a node to them
---------------------------------------------------

number of hashing functions and replication levels are complete;y different deom each other.!
---------------------------------------------------


So like 3 hash function per shard to get three replicas across Ring?

Replication levels: 
- more than 3 replicas are wasteful

- master-slave: what kind of architecture do you have?

- highly available. or eventually consistent. or highly consistent - but never seen highly consistent.
- even banking systems: eventual consistent. 

what to do  - if extra available?
- wrt shards? replicas?

how to add or remove machines? 
what happens if a machine dies?

questions:

1. how to add or remove machines?
    - if extra machines are available - what to do.
    - what happens if a machine dies?


so now we are the orchestrators.

what do we do for these 3 scenarios?

1. extra machines - simple consistent hashing? 

possibiltiies : 
A) distribute equally amongst shards?
- create a new shard - if replication level can be maintained.
- add extra replicas
- leave them idle: keep it for peak time - or monitor and help out.


tackling option (A) - 
1. we can do this - but we create N/R extra shards

Better algorithm : 

***- replace downed machines first. 
- if extra machines leftover ->  again new shards -> rest are idle
--------------------------------------------------------------------

I have a question
when we created a new shard ! from how the data is spitted between machines (replicas) - using consistent hashing -_-

I have a question
when we created a new shard ! how the data is spitted between machines (replicas) and from which replica that data is moved and how the sync happens on other replicas ??

keep number of machines in ideal state - this number should be equal to nmber of shards, so that if some machine from shard goes ideal, we can replace it

dont immediately create new shards if you have enough machines..

how to calculate ideal valuye of X? 
X = how many machines shall we have as backup?

Google's bigtable - first nosql

using probability and back envelope calculations:
choose between X backups - and then creating new shards, with their replicas

what about idle  machines? do you put them to work? do you keep them idle?

- NO! not idle - use as extra replicas but with extraz obligation to backup if something goes down

do we keep replica of orchestrator as well ? it may fail as well

replica vs hot copy:
data goes away - stateful machines - make replicas.
orchestrator - goes down - hot copy

LB, orchestrator, etc


there should be multiple orchestrator.. how do they communicate between them..

no - only one, and maybe hot copies



To: Everyone
10:33 PM
what if when we need to remove shard in case of multiple machines got failed?

How is data moved?
1. when a new shard is created? 
2. A new read replica is added?

backup machine will come up in few sec.. if we have 100 tps.. how do we handle that.. could you please discuss this..still not clear

on orchestrator - hot copy concept - the orchestrator is NOT handling the requests - it is completely seperate from the request cycle

its only managing new dbs, migration of data etc

who will be maintain routing table?
Load balancer for app server, and from app-server -  the db client does this - like mongodb client

Seamless shard creation...

1. new shard -> 0 data -> cold shard
migrate data from old to new machine
--------------------------------------------------------------
suppose added new shard via consistent hashing - but no data - so NOT directly right?

so now - BEFORE consistent hashing - we want to MOVE - not just COPY the data..

so stepwise - 
1. dont really add shard - only orchestrator knows new shard is to be added -> not the DBs themselves or the appservers etc

- so first we do a simulation - we simulate what users/requests are going to this new shard?
- so the range of hashes basically - are detrrmined here.

- then WITHOUT telling anyone -> simply copy the data from the simulated calculatiions (hash servers etc) ->say 15mins

- now the new shard is warmed up -> only the data BEFORE the shard simulation started has been copied ie prior to these 15 mins. If there were write requests in between - new shard is missing that data!

- now take it online -  informt he DB clients. add it to the real hash ring.

- now we need to catch upto those 15 mins of data! 

-> we have 2 choices:

- highly available: simply start serving the requests - but we might have SOME data that needs copying - which will happen SIMULTANEOUSLY (without the delta updates)

- so it will serve stale data for some time - and after say 1 minute - we serve all requests with latest data

or we can bbe ....
- Highly consistent:
reject all requests for the time taken for the DELTA update!
- reject BOTH the read and write requests
-------------------------------------------------------------------

AFTER this is done - you can simply delkete the data from the old shard !
-------------------------------------------------------------------

it may give 404 for few read queries when new shard is added as few items are yet to be copied to new shard ?



the data is huge, how we identify the delta ?

concept of WAL:
------------------------------------------------------------------
- all the DBs unconsitionally maintain a Write-Ahead Log (WAL) - so simply copy this file - and execute these commands on the new shard!

- Ajit Kumar
To: Everyone
11:15 PM
like commit and rollback. before commit things are temporarily updated, not acually
------------------------------------------------------------------
- for rollbacks, transaction, power failure or server down, plus improve write speeds

- due to ACID properties that DB offers... -> also things liek SQLite simply write to WAL to maintain faster speeds!
-----------------------------------------------------------------

Backup machines - the idle machines...
estimating num is required. but complex!

so some DBs (dynamo db, cassandra etc) use completely different approaches for maintaining backups, replication levels etc

they doint have master slave replication at all!
-------------------------------------------------------------------
Muti-master replication!

Multi-master: when every ,machine is a master...

- get rid of shards completely - instead - on the hashing ring - every machine gets a spot, not every shard! so machines are basically not grouped into shards... every shard is exactly one machine even if replication levels>1

- how to maintain replication levels? (say RL=3)
simple consistent hshing for a request/user will yield 1 machine...what about two more? where to?

- so the next 3(or RL) machines on the ring store this data! using consistent hashing!
- if all 3 machines die - you loose the data!
so calculations ar eimportant here!
-------------------------------------------------------------------

- now - M2 is NOT a replica of M1 - its just additional data on M2
- moreover: M1 has multiple spots on the hash ring - so M1's data is stored in multiple replicas onthe hash ring - on whatever the next 2 machines are!

- each machine then has partial replicas on the next 2 machines on the hash ring - which means various machines due to multiple hashign functions!

Now, when a new server is added - just add it to the ring - no shard management is required.
---------------------------------------------------------------

It also provides somethign called "tunable consistency" - given a partition.

It uses 2 variables: R and W, plus RL (replication level):
 
- R: read is succesfull if ATLEASST R copies return same data
- W: write is succesful if ATLEAST W copies return same data
----------------------------------------------------------------

So basically every read request is done to R machines - and if data is not same - merge it for the latest

Similarly for write request!
-----------------------------------------------------

Note - even tunable consistency will eventually BE consistent! using cron jobs or soemthing

Effect on tunable consistency?

As R or even W increases, consistency goes up!
- if R is higher: it becomes less available - same reasoning as CAP
- if W is higher: same!
----------------------------------------------------------------------

what is consistency? we mean - every read has to fetch the latest data - and NOT - every server having the latest data!
------------------------------------------------------------------

If R increases without increasing W: read from many - but possibkle that all reads from stale servers

If W increases without increasing R: might write to many servers, but might read from stale servers again!
-------------------------------------------------------------------

BaLANCE? R+W>X : EVERY read/write will have ATLEAST onme server pointing to the latest data!
-------------------------------------------------------------------
by the pigeonhole principle*** in DSA



At R+W=X: X is the RL
still might get inconcsistent data ///

also replication factor is the thing that ensures eventual consistency


consistent hashing is ONLY required when we have state.
Not on stateless ofc

App servers are mostly stateless - simply RR from LB

But on DB layer now - we need consistent hashing done by dbclients
-------------------------------------------------------------------

If servers cant be stateless - they might need consistent hashign too!
-------------------------------------------------------------------

no sequence to which sdhards - it will simply choose randomised shards for reads and writes

cassandra, hadoop maybe mongo have their own orchestrators


LSM table, SSM tree

if we have replicas, why not one in replicas make a primary, instead of copying to next clockwise node?

in tunable consistency , if different servers gives different values , which one will be picked

so orchestrator just for managing the server like add or remove based on some algorithm but data copy job is not part of orchestrator ?

- Are Banking transactions also Eventually Consistent? Or just non critical aspects are eventually consistent? Do they use some insurance mechanisms to afford eventual consistency?

so orchestrator just for managing the server like add or remove based on some algorithm but data copy job is not part of orchestrator ?

YES - 2 phase migration is all part of orchestrator - but not exactyly copying  - only the managaemnt - some other job reports back to it

Banks LOG our data - so eentually they get to know stuff - but maybe not immediately



- Criket Match Scores Website Usecase: Should server update the clients about the change in scores, or should client poll updates repeatedly?


- Merging/Choosing Reads happens on basis of timestamp?
- Is this R,W also called Quorum in Cassandra?

Are these buffer machines then decided by #9s in Level Of Service.

Are replicas like different connection url to different DB servers, with some master slave hierarchy?