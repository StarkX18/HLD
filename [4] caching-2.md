Agenda:

1. invalidation and eviction
2. local caching - case study 1
3. global caching - case study 2
4. case study 3 - fb news feed

Recap:
1. consistent hashing - how
2. caching
3. CDNs

most applications are read heavy. 
- caching layer is added to make this process faster.
- challenges: caches are not source of truth, its the db
- caches are small in size -> cache misses happen -> need to maximise hit/miss ratio
- proper cache design, cache policies etc maximise our ratio

keywords:
- cache invalidation, eviction, policies etc

Cache invalidation: removing/refreshing stale data

TTL - time to live 
1. store data with created_at, and expiry time.
2. if t<TTL, fetch from cache
3. else fetch from db

cache miss - either cache hit, followed by expiry time invalid, fetch from db

or cache itself removes data after fixed time.

which is better? wait for user request. 
- no needless requests
- extra space

Are stale data negative caching?

but some data might not be time dependent right?

TTL:

1. pros: simple
2. cons:
- TTL must be set aptly
- if too small -> might not be usable alot
- too large: stale data

So after TTL get elapsed then we would trigger expiration?

but before the expiry time if someone updated the data in db then? in cache we have older data but in db we have updated data and that we need we can handle this

eventual consistency using TTL in linkedin feed.


https://www.linkedin.com/posts/agarwalpragy_hld-class-eventual-consistency-demo-activity-7031652033099149312-I5fD?utm_source=share&utm_medium=member_desktop.    :P

but from user perspective it seems its not right thing then what could be the right way to do it ?

see lecture before this!!

LinkedIn post:
 eventual consistency instead of strong consistency.

LinkedIn's servers time to live server cache time is prolly like 5-10 mins!

For all CRUDs

Cache - write through - concept:
- any updates must first be written to cache , receive response-> followed by db write, receive response -> followed by response to user.

- looks like cache is writing to the db - but thats not what happens..

- what in case of faliure: retry and repeat the cycle, from the step it failed on!
- after x retries, return failure response, and revert previous steps too ie. remove from cache

why write through?
- cache data - feed, etc can never be stale!
- reads are super fast! as every cache hit will return directly from cache


why NOT write through?
- writes will be much slower.. as both cache and db done at same time
- hence makes it suitable for a read heavy system.

- majority of systems are read heavy, 90 percent!!!
- example - likes on a post - write heavy, views etc

so there is no issue of eventual consistency in write through cache?

for write through approach, wont the cache size need to be very big ? since all writes are stored

here we need to have TTL in write through cache

if we follow this strategy (update expired data only if a data req comes), that it is a possiblity that at some time we will have lots nd lots of data even if it is not required..

what happens in a cache miss?
normal scenario wirth db read

size of cache would be same as db right? so it would be costly

can you explain how read are faster here  ?

cache hit to miss ratio is a tipping point so design it properly

Cache write around concept:
- writes happens directly at db
- cache might be stale, more frequently
- it can be updated via cronjobs, with recently updated data or something like that
- push based mechanism!

In this case we might lose on most visited data as updated data will take the cache space 

cache write back concept:
- write to cache and return success to user
- cronjob flushes this data to db periodically
- write heavy systems...plus consistency isnt important
- messaging apps? NO - v-day velentine crush -> consistency is important here!
- tracking clicks, analytics etc -> trend based etc things
- for views - twitter/insta might use, but views - likes wont matter!

Write Around - so basically if multiple people writes a data and only few people sees the data, then it is a scenario for write-heavy system???

Can we save to both database and cache in asynchronous manner using some events?

only write trhough is synchronous here, other two aint

Cache eviction:

- cache is limited in size.
- what happened if cache could be as large as db

policies:
1. FIFO - queue
2. LRU: most frequestly used policy
       - hashmap+heap or hashmap+DLL
3. LIFO 
- most recent reads are evicted first
- use cases? -----------------> DIY!!!
- reading groups - one piece example
4. Least Frequently Used:
- intuitive

local vs global cache:
-> local: every app server has a seperate cache
-> global: a whole seperate cache layer with multiple servers for caching -> but exact replication of caches

Linkedin post demo that you showed - discrepancy was because of local cache ?

Is Global Cache slower than Local Cache?

case study:
online coding platforms!
= leetcode, spoj, scaler

- see the flow diagram
- browser -> dns -> ip -> LB -> app-server

- now what does app server need to evaluate solution?
- user metadata, problem metadata,  code submitted and testcases/input files and output files for results
- input files~500mb , output files~500mb too

- where do you store these filkes? not db coz it doesnt store large data in single column
- but not optimised
- used file storage system: s3 -  simple storage service - no sqldb blob storage for storing large files on the cloud! 

- read/writes might take huge time!
- say 1Gbps and 1GB file -> not 1 sec but 8 sec to read file

also, each app server has hdd space - 
go to s3, download file in hdd -> use for further cases -> using hdd like caches!

- change to the testcase? reasonably static, but subject to change too..

- write through, or write back? we have multiple caches across different machines so write through might not be the best choice.
- 100 servers, 1gb file -> 100gb of traffic to change a testcase :)

- write back/TTL : some period during which data will be stale
- too short ttl: too many cache miss
- too long ttl: updates wont happen - might get bothered by wrong testcases in a contest


really cool links:

https://gist.github.com/jboner/2841832
jeff dean! best engineer in the history of computing

updating test cases:

local cache?
- at every write, push change to all servers
- write through cache basically
- multiple caches, with each server having its own cache
- write back cache - as discussed above bad solution...

global cache: bad solution.

- multiple app servers -> no local cache on anyone,
a seperate cache layer.
- a lot of network traffic! 
- plus a lot of data as all will continue to hit.

- too much in-memory size will be really costly:
example 32GB RAM = around 70 files! 

- why not HDD: csan't serve all app servers as 100GBps read speed rqd!

- even top of the line SSDs

Store it on client side? NO - network traffic on a much slower network!

actual solution:
can we somehow store some metadata in db which tells to cache evict?

prob_id, s3_file_path, output_s3_path, ip_updated_at, op_updated_at

request -> app_server -> local cache (HDD) -> get metadata for problem from db ->if stale -> fetch from s3 -> else -> serve

so a write through solution??? DIY

test case changes:
1. upload to s3
2. change metadata db -> all caches refreshed upon user requests!!!!!

what changes: 
- now data to be fetched is less, 
- all caches are updated simultaneously

2 more case studies left!

in global cache , if we have multiple cache servers , how they synchup ?, give some  example where for use case of global cache !????

Is it possible to have a system, with global cache having multiple servers, having different data on them.
How things will work in this case?

if metadata matches with the hdd cache then cache hit?otherwise miss and get it form s3

https://try.redis.io/
https://university.redis.com/
https://redis.com/blog/
https://docs.google.com/document/d/1k4nzubvtX_yLctUT4VWK8ZJt4KCcOEdRJdxQgWCaiU8/edit

is it right understanding that there are much more approaches of updating cache other than writeback, write around and write through - like the metadata file timestamp approach that we just saw ? based on the requirement

what was the need for multiple layers of cache? l1,l2,l3 cache layers

more methods of caching-
metadata file approach etc

registers -> flip-flops
ULSI - ultra large scale integration
L!->L2-> etc

the metadata soln is still better but how about using write around approach

I had asked this question in the last class but missed this in the doubt session- if app server contain business logic and retrieve data from db and the business logic is same, then why do we need multiple app servers? 

the metadata soln - write through, write back or write around? 

1. write through: no stale - write to cache and db
2. write around: cronjob updates cache using db
3. write back: update db using cache

metadata - s3 is source of truth
cache had no stale - because metadata was updated

DIY!!

instead of storing metadata in db, file names stored in s3 can have timestamp?
instead of storing metadata in db, file names stored in s3 can have timestamp?
- possible! - store metadata in s3 but again, call to meta db is still required!

if file would be extremely large which is larger then hdd what we could use in that case ? 


map-reduce: for splitting file across multiple servers -> like apache spark, hadoop etc

are we going to cache every requested problem data in HDD? what is cache eviction strategy if HDD gets full?
run cleanup jobs on files/folders etc..eviction is definitely rqd

how querying in Amazon S3 file storage is faster than normal database like postgres when it comes to big sized files???

for s3: file storage, like NTFS, FAT, ext4 etc -> store particular index, fetch it

DB are optimised for querying data..
B+ trees:
- storage always comes from hdd
- in case of hdd -> continuous vs random is very diff - localitty of reference..

- disk spins at 6k rpm - sequential data is easier.
disk seek operations are so much more harder - ie moving the reader!!!
- how db works

s3 - same things but something diff - see!

Are cache always inmemory ?
no

How do we(Application) will communicate with Cache ? 

For different type of data we use different strategy on same cache server? yep

in case of ttl
if we try to update cache entry when it is requested
what happens when cache entry is not requested? cache will still have that entry forever, right?

Does cache eviction kick in only when cache is full or strategy of certain percentage being reached also can be used to kick in the eviction strategy?

how TTL is not suitable for onlnie contest? why global cache is not suitable? why write back cache is not suitable?

does load balancer also uses cache? if yes, what type of cache generally?

Why can't we store the metadata in the appserver's local cache?

How the timestamp in metadata in database is updated ?

why app server do caching randomly ? if we want to read how we will ensure that we will get cached data ?

On Caches - What if we expire a random sample size say 0.3% via LRU that way we don't have to touch all the entries in cache where eviction is applicable?

what is ideal size of cache?

in which scenario, write around is used? write heavy or read heavy?

irrespective of user read or not, db updates the cache..whereas in TTL, it only happens when user identifies its expired

Can we save to both database and cache in asynchronous manner using some events? In this case, won't we have `write-through` cache and `write-around` cache? 

is any chances of cron job fails due to that the cache will not get updated

are writes on cache also faster than db? in case of cache miss, we wont be reading from cache again right? only db rread, + cache write, so lets say 11ms ?

if data size is more than memory  cache size in case of write through then what will happen ?

what if cache crash after write through and db is also not updated ?

Read Heavy application can have a peak write time, when every one wants to write. Do we still design such momentarily write heavy apps also as read heavy? How to overcome slow DBs in such slow writes.

size of cache would be same as db right? so it would be costly

How write through cache works in distributed cache?

What would happen if user tries to get the data and then got the cached data but it is not yet updated at all in DB and retry is still ongoing.

what if write to DB is not success? Cache is updated but no update in DB

Can't understand diff between proactively purging  Cache and waiting for Cache Hit.

Cache we will only update when Request made (not when data TTLed right)?

May be just running a cron to periodically purge Cache at TTL might take time though! 

But apart from that, in both approaches, Cache (upon request), fetches truth from DB.

one question TTL cache size will become very large  right ??

So if the link has the request to new message that the distributed cache don't have the information it gets from server. BUt for some message that it has, It fetches from the cache itself???

so when does this call happen, when the time expiers, the cache automatically call db or when the user makes a call and find the data stales then it makes the call to db ?