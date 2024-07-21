Recap: Caching - 
1. TTL
2. Cache invalidation
3. Write through, write back, write back cache

Agenda:

1. Invalidation and eviction
2. Local Caching and Case Study 1
3. Globl Caching and Case Study 2
4. Case Study 3: Facebook News Feed

Cache is not the source of truth - DBs are.

However in certain caches if DBs are small - cache the whole DB!

important thing about write through:
- strong consistency + read heavy

- write to both - if one fails - other too.
- otherwise - return failed response - retry - and revert changes

- so a system like irctc - its the best chocie? in irctc - we DONT end up using a cache!

Why we use global cache + write through combination generally....

can we write a sync between database and cache to keep it consistent, instead of doing it by App server?

yes - write back. or write around cache it becomes.

can there be a chance that database and cache be out of sync at any time in this case? For eg in case of concurrent writes?

 in write back cache: it kinda acts as the source of truth..

media net ad tracking!

write back - high write throughput, cache as the source of truth in fact.

-> but - this is also low consistency


Can concurrency becomes an issue in write-back cache?
We might need locking for this right?

No its a pure write - theres no same write too

Also we don't care about consistency here

suppose iwant ot refresh data every 15 mins and consistency is important which one to use

Loose coupling  -> in algorithms as well as hld -> layers of LB, cache, app server are independent.
Gambling! - on a side note.

in which scenarios we go for write  around cache

how do we cache files like audio or video?
- CDN - not in-memory

- CDN stores Large files!
- Why does data reside in S3 and NOT in CDN? It IS capable and we CAN do that!
- CDN can ALSO persist for long times!

- CDN is used when data is to be sent to USERS - across the globe - no use here!

Does this mean we can use CDNs instead of S3 for such cases?

Cache eviction:
1. FIFO
2. LIFO
3. LRU
4. LFU

local vs global cache: when, why, where?

is cache different from CDN?
- CDN not equal to cache! CDN in FRONT ie BEFORE the load balancer!
- cache sits behind it

Netflix uses cdn/s3. How user watch the videos? netflix server caches the video from s3 ?

today's class: contest leaderboard!

-> 30k people, 100s solution every sec
-> ranklist? - true ranklist updates with every submission!

what do you think?

write-back cache - <userid> as key, <score,time> as value

we want a write-heavy + fast system (hence cache) - which is also faster to read than db -> so write back/around cache

local vs global?
everyone should see the same RankList !

Starting today's class - finally! :P

Designing the contest leaderboard...a LOT of traffic and HUNDREDS of submissions.
- ranklist updates after every submission.

How can we calculate ranklist?

user id, times submitted, time taken, status of problem etc..

how frequently does the ranklist change?
- immediate consistency is not important.
- updates few times every 1 second or
- updates periodicallyt after small interval ~ 15 mins

twist n turn

people will see different ranklists? but updated every minute -> plus not a lot of kv pairs -> 30k around ius the largest in india

but each app-server will go to db and recompute it.

TTL vs write-around: 
TTL leads to lots of db recomputation every minute
write around - one recomputation and pushed to all

Deciding local vs global cache:
 
- local cache: whenever submission happens: other caches become stale
- to refresh other caches: TTL or write-back cache

- take write-around (push) : cron job computes ranklist periodically (1 min),  and updates ranklist in ALL of the caches...

- assuming a scale of 30k coders, not a huge load.

- take TTL (pull) : after 1 min - each applist ranklist expires. so you recompute ranklist, due to each appserver, and then dump back etc

- notice though there's not much difference between both theoritically but it REALLY makes alot of difference!

- first is faster!

Global Cache: better solution
- every 1 minute cronjob, updates global cache!
- thus even better than write around

-------------------------------------------------------
- why? 
- in write around - we update each appserver's cache, but in global cache, we send data to global cache only
- also, in cronjob's update to local, we HAVE to send the ENTIRE ranklist to each server

- any diff in how we read data? prolly not, both would be paginated..

does global cache crashes?
yes.

In real scenarios, we can have multiple contest running in parallel. And each contest has leaderboard. So in this case, are there separate cache maintained for each contest leaderboard or it is common?

Is global cache accessed using endpoint APIs ? jus for understanding purpose


In real scenarios, we can have multiple contest running in parallel. And each contest has leaderboard. So in this case, are there separate cache maintained for each contest leaderboard or it is common?
If common, how pagination will be maintained in this case as data might be mixed.
If separate, how this is created and maintained as we have single RAM?

but in glovbal cache we have all data after compute right>?why we reuqire pagination

Can we say Write Around caches are good for Eventual Consistency scenarios, just like WriteBack Caches good for Analytics and Write through for Read Heavy systems.

will we study distributed cache ?

I have 400 MB cache on user machine which used to populate master data, however any deletion in master data, which approach to use for updating master data, instead of re creating cache?

what happens if a global cache crashes?
- recompute ranklist and udm here

so  app servers access data page wise not a qwwhole?
is there any db which is better suited for this kind of purpose? like maintaining a sorted column.

How does the app server know that in which cache it should look into?

Session management can be a use case for global cache right? Yes

is there any db which is better suited for this kind of purpose? like maintaining a sorted column.
- every db does this
- however there is a cost of updating the indexes,
which depends on how the writes are happening and what index it is...

kind of queries to cache server//

1. given a user handle, find rank
2. give me ranklist for page x.

redis:
single threaded, <k,v> store + various data types

-> lists, sets, ordered_sets, custom datatypes etc

King of kings in cache: Redis :)

- give me ranklist for page X
- given a user handle, find rank

redis does this..
store strings, bools, ints
lists, sets, sorted sets etc

Its coded up in C! not C++!


https://try.redis.io
redis.io/docs/data-types

homework:
university.redis.com

fxnality to cron-fetch data and persists to disk!

https://try.redis.io
redis.io/docs/data-types
university.redis.com

-> no locks, no concurrency stupidity
-> why single threaded? 

Any significance of Single Thread only in Redis? To avoid inconsistencies from writes, if done from different threads?

why single threaded? 
- consistency due to NO concurrency
- implementing async code in C is hard prolly :)

To compute the news feed, what data do we need?
- posts by - me, friends, people/pages you follow, groups
- comments on a 3rd person,
- posts by others, that your friends have interacted with
- recent data
-----------------------------------------------------------------
- user table, id, name, email... etc
- userId, friendId,
- postId, textId etc
- does this really happen in RDBMS model - yes all big apps use RDBMS!
-----------------------------------------------------------------
- not a graph or even nosql DB!

If you want to capture news feed.
- select * from posts where user-id = <>, order by time desc limit x offset y
- select * from posts, join user, user-friends, user-groups, news_feed where <>

does all user data fit on a single machine? No, sharding required.
1. Sharding key? for profile page?
2. what is my sharding key?

shard by user-id?
- User DBs : how many posts will a user make in his entire life-time?
- posting once every hr
- all yrs for past 20 yrs
- etc

-> 100MB of data per user

How to retrieve data for news feed?
If sharing, getting posts from multiple shards, and combining - bad for network traffic and db!


CANT cache since computation still requires a ton of feed! => 
1. how to compute? single news feed = multiple users who are friends of me
2. cache user => news feed
----------------------------------------------------------------
1. billions of users, news feed has 100s of posts
-> each post is replicated in MULTIPLE feeds!
=> cant we use simply post ID??? no, multiple shards!

2. fan out updates -> every post has to be copied to all the feeds!
3. for every user, you are storing the actual news feed -> very hard

Can't we have mapping table of post_id and actual post data `in-memory` itself? This will reduce the reduandancy... isn't it?

how data is scattered over shards for one user...but you said that one shard can store data for approx. 10k users

can we have background task which keeps on updating feeds for a user, and profile page will retrieve from feeds ready cache?

Like Metadata approach, can we store in DB the friend's UUID list as metadata. And then fetch selectively from only those DB shards the friend's activities? Each DB is sharded on uuuid only, containing their posts and comments.


how can we guarantee all the data for user fit in single shard?

Like Metadata approach, can we store in DB the friend's UUID list as metadata. And then fetch selectively from only those DB shards the friend's activities? Each DB is sharded on uuuid only, containing their posts and comments.

to handling the sharding of database we are using consistent hashing right?

difference between s3 and cdn?

In news feed use case : so every 15 min , we are updating all replicas of derived data set ?
isn't  that  huge ?
We are querying user db and then updating replicas every 15 min.

in news feed, if once we have already seen a post, we dont see it again right? if we refresh

Apart from Blocking Single thread, and parrallel threads there are alsp Async single threaded architectures? Like NodeJS seems to be async, concurrent and non blocking. So Redis is more like single blocking thread, or single non blocking thread.

How does the app server know that in which cache it should look into viz. Local or Global?


incase of leaderboard. Do we have to store only userid and score or we can store a object with relevant user details  in sorted set ? 

I heard somewhere that even dynamic content is served by CDN. Is that correct?

computations are same in both cases write around and global cache, how global cache is better i did not get it
also is network traffic/calls between cron job to cache same in both approaches?

the cron job that populates the derived data store should be run very frequently right (like perhaps every minute) to make sure that the derived data store is updated with new content as soon as posts are created? 

apps like netflix use cdn, right?

This qn is not related to caching, but would like to know how the test cases and code submitted are combined and run, does they run in separate process and result of that process are responded by the app server?

also, how does fb manage that a user has already seen a post and wont show it to that user again? 
i understand that it might be more of a implementation question rather than a system design question

suppose I want to fetch the ranklist for the page a user is on -> I think this does happen in scaler itself. Is there some way to directly get this from redis -> and avoid two cache hits ? one to get user rank, second to get the page for this rank.

the only pro for local caching in local vs global cache - will be speed of reads/writes right? otherwise global cache would always be better? Or is this too simplistic ?

serving big file from app server would not hit the server performance??

Q). Can we use WriteThrough cache between App server and DB for metadata queries too. If S3 updates, then we write through this Cache and update DB as well.

How to update in-memory cache  all servers with every request from app server?

in case of write through cache, is it guaranteed that database and cache always remains in sync? For eg in case of concurrent writes, what if the later request completes first while updating the same key

But there are many UPI application which could share loads 

how can be the user and post mapping in bloom filter soln?