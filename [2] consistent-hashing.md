0. consistent hashing - contd..
----------------------------------------------------
1. appserver layer
2. caching
3. invalidation and eviction
4. local caching


1. what id the LB goes down?
2. which machine to send request to - by LB? -> whats the routing algorithm?

Where to send the request - on LB?
- where relevant data available!
- can we store all the data on a single machine? no its split! - concept of sharding
- backup is a whole different story! for later...

how do you shard the data?

- do you split it randomly?
   - could do that but with cons: no idea which data is located on which machine.. and we want to read the request from single machine only - not query multiple ones


requirement for deli.icio.us : lets narrow down:
- for a user, get all bookmarks
- not for a bookmark, get all users!

-> something like a hashing algo could be used? <row id-> machine id>
-> hashing can be used based on user_id


does LB know which requirement - on which server?
-> server crash? (# servers change)

assume replication or black box for now
-> but suppose num servers changes: hashing table changes!

-> if hashing fxn: simple mod -> tons of problems as rehashing happens!

here the problem is not exactly the rehashing right? its the data movement that will happen due to rehashing?

YES - data would be moved to hashed servers

what if we apply mod on server name which can be numeric?

here the problem is not exactly the rehashing right? its the data movement that will happen due to rehashing?

but would there not be scenario where there would be too much load on one of the servers if any server crashes ?

mapping table:

if server spun up: update some mapped values randomly
if server down: map only those to other server.

con:

1. if this LB needs to be replaced, this data needs a seperate backup

2. this table might be very large -> might even need disk; which in turn will cause huge problems -> you dont want another db in LB bitches!!!

If we can somehow know which will be the server that request of U1 will go to, then we can do copy when the data comes in.

These issues are only when we have stateful server but in case we have stateless servers, any of these technique would work right?
Is my understanding correct?

range based approach:
- 1-100:s1, 101-200:s2 and so on!!

- we know which server data on. yes!
- server crash: yes - only those rows are moved to other server
- server added: supposing new data was added -> yes, will help for those users
- but old LBs not helped - load not distributed uniformly!

- prolly unequal distribution which will only grow as loyal users keep using our service more, and rest keep moving...

final solution: consistent hashing:-

- hash both the user_ids and serverIDs to figure out which user maps to which server -> set of values which are very large range! long int --> pow(10,18)
-> take modulo to keep everything on one ring
- sha256, md5 etc

- now once hashed: placed on same ring.

- good points:
   - server addition is handled beautifully!
   - one server crash doesnt propagate until load becomes too much!

- problems:
- again load distribution is a problem as servers aint equidistant from each other!
- magnified by problems of server crashing
- once enough load arrives: domino effect or cascasing crash might take place!

current problems:
1. load's equal distribution
2. cascading failure
-------------------------------------------------
proposed solution: instead of 1 hash fxns: use k hash fxns!

-> every hash fxn gives one spot to our server!
-> once a server crashes - it is distributed amongst various servers! - due to more hash fxns

# server down problem solved!
---------------------------------------------------
server spun up -> even better -> more spots on the ring -> more load distribution!
---------------------------------------------------

Implement consistent hashing:
1. multiple hash fxns
2. now put on a ring:

use an arr - n servers, k hashes -> insert into sorted array:

when request comes: hash(user_id) -> search in this particular array -> and move to next server available!

-> array saize: nk

Are you clear about the following things related to consistent hashing? (choose all that you understand)
229 responses from 40 users
A
Load Balancer needs a way of mapping Request to Server
14%
B
Mapping randomly won't work. The Server needs to contain the data of the User
14%
C
Mapping techniques like mod, mapping table or range based have their cons
15%
D
If a new server is added, we want it to share the load of existing servers
16%
E
If a server crashes, we want its load to be distributed across other servers
15%
F
If a server crashes, only the users previously going to that server should be re-routed
15%
G
How consistent hashing solves all the above issues

consistent hashing doubts! :

assumptions:::
1. multiple app-servers - assume data is stored in app-servers themselves
2. data is sharded ie partiotioned across multiple sources -> sharding is a special ccase of partitioning where it means multiple MACHINES
3. LB gets requests from users and has to send them to app-servers - which HAVE the required data - otherwise multiple requests created!

-> now data and request-routing have to be maintained!
-> + health check/heartbeat -> how many servers alive
-> maintains them and uses consistent hashing as discussed.
-> multiple hash fxns -> server up and down do not impact! however data and requests will have to be migrated upon the rehashing scenario of servers count changing... 

doesnt user id and server id need to be normalised for consistent hashing?

so that hashes are evenly distributed?

Same question, if the server is down what will happen with user data?
- k=16/18 generally
- marginal benefit is most at this point
redundancy and replication!

also why dont we simply divide the circle uniformly wrt num_servers

imp distinction:
every user when hashed, has only one spot
every server has k spots


so now the only job of LB is round robin? which is (++ctr)%(num_servers)? isnt that too less of a job for a whole server??

A few questions from my end -  
1. the data present in the crashed server has to be copied in multiple other servers, right? Won't that be a problem?
2. on what basis is this array sorted? 
3. hash function for user_id will be diff as compared to  hash functions for server_id?

how to decide most efficient value of K?

How to decide good hashing function? is there any real-life hash function example that you can provide that we use in consistent hashing?

aren't hash values unique based on the IP

how does the hash function makes sure that the hash value <=10pow18 ?

How does it find the next nearest server for a userId hash?

Will it still be possible to have all the bookmarks of a user, even after hashing (using %), in a single machine? Can't it just overflow and need to be shifted to some other machine?

Will it still be possible to have all the bookmarks of a user, even after hashing (using %), in a single machine? Can't it just overflow and need to be shifted to some other machine?

- well for the bookmarks - its highly unlikey
- but there might be a case where user data might be huge enough to not be stored on a single machine itself!!!
- within the user data- sharding might be required!


How to optimize sharding or partitioning for multitenancy, for instance if we want to distribute data from multiple tenants(or users) in a Saas-like software?

Here by server we mean Db server right? Cuz all application server can be connected to one DB

So sharding is applicable on database servers only right ?

multi-tenancy and sharding:
- in case of multi-tenancy, sharding becomes even easier!
- multiple tenancts -> every tenant becomes a shard -> tenantId can be used as shard key!

So sharding is applicable on database servers only right ?

- yes

is sharding similar to partitioning?

sharding is a subset of partitioning. 
- when across multiple machines - sharding.
- horizontal and vertical partitionaing and sharding too
- partition serves some other purposes too -  like increasing write throughput

so all the data for single user should be on single machine or that need to split as well?

suppose if we are requesting some data in machine 1 , how our request will go to machine 1? or it will try all the machine

will we have sessions on sharding ? NO