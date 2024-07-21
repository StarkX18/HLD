Agenda:
1. nosql databases: common types and 
2. when to choose what
3. manual sharding in sql vs auto-sharding in nosql
    - multi-master
    - replication

are we discussing master less/leader less also today (like dynamo)

Will you explain P2P Architecture?

SQL vs NoSQL

sql:
- ACID
- schema

nosql:
- flexible schema
- ACID useless in sharding


Types of nosql:
1. key-value:
     - key:string
    - value: any thing - redis very powerful here

- completely schema-less
- only possible query-basis: key
- no indexing - nothing fancy

example: firebase, dynamodb, redis, memcached 
- not allow json
- only string to other mapping - not like json store - read more!
- why redis? is it a cache or db?

----------------------------------------------------------
2. document-db: json stores

- every document has a universally unique id - uuid
- loosely defined schema
- supports queries more complicated than key-value
- EVERY attribute ALSO has automatic index!!!
- smallest unit of any CRUD is document id

- but in dynamo we also have sort key right? read
- can we qurery common things...
- but in mongo , I remember we have to create them manually - indexes
- does elastic search indexes all keys throughout the json?
- how is the json stored internally in server ?
- what will be the benefit of sorting keys? key is ith what we partition, but sort key acts as a secondary local index


example: mongo, docudb, elastic search!

-----------------------------------------------------------------
3. column-family db

------- others - not to be discussed --------------------
4. file storage - 
5. graph db - knowledge management - not even fb
- basically optimise graph ops
- no fb, google maps, uber etc - use graphs, not graphs db
-----------------------------------------------------------------


https://www.mongodb.com/docs/manual/tutorial/getting-started/ 

https://mongoplayground.net/

how do you shard these DBs? they are supposedly easier to shard! why?

No ACID properties - so yeah it is.

1. key:value - based on key
on indexes - we have B+ trees ->
- document db index stores not only key and value but also document id  
2. document db: document id
3. columnar db: based on row id - and you can choose what the row id is!

any query - it stores indexes for all keys, as mentioned earlier. also stored is document id, and the value -> so it becomes easier to shard


Piyush Mishra
To: Everyone
9:34 PM

Doesn't MongoDB gives Transactions feature: https://www.mongodb.com/docs/manual/core/transactions/


basic premise: every document/key is independent of each other: 
- thus each document_id / key can be stored indepemdently.
- all data that needs to stay together should not be far apart

column family databases:
what are they?

- seperate column like  posts, comments, messages and correspondiing to each column vamily - there is a timestamp and a corresponding value!
- eahc column ois basically a list of values + a timestamp - NO other attribute!
- each rowId can write to specific cols

- why nosql? because corresponding to any row: there might or might not be correspoding rows in other column families.

- example facebook messenger: user id can be row id.
- its also the ideal use case for sharding = the user id
-----------------------------------------------------------------

available queries:
- fetchMostRecent()
- fetchByPrefix()
----------------------------------------------------------------


Main concern of tries?
-> space optimisation 
-> prefix match

Why columnar db has prefixMatch queries: no tries - itas a side goal: so sorted list

-> examples: hbase, cassandra etc

can we have what ever databtype in columns databases

checking out different databases:

#1  twitters' hashtage data storage
-> context: for each hashtag, give me the top K(10~50) most recent/popular tweets
-> upon scrolling down - more results - paginated data!
-----------------------------------------------------------------
what to use?

- columnar : why? most recent query
+ popularity: sort and paginate



comparing different databases:

0. relational
     - cant handle scale wrt sharding - manual sharding can be done but might be costlier!

1. key-value like redis?
    - key = hashtag, value: [array of tweets] : too 
       much data - on max traffic like diwali - 100 
       new requests/second or more
       or 
       array of tweet_ids -> 2 db calls
2. document like elastic search or mongo
    - uuid + hashtag + tweets[] sorted by 
       timestamp lets say
    - rewrite whole doc on high traffic - since 
       whole docs are replaced
    - if every tweet is a seperate doc -> most recent 
      tweets -  very easily possible, pagination too
    - issue: popular tweets? assign a score based 
       on views, likes, comments
    - another issue: filtering based on multiple 
       columns -> aggregation pipeline -> all 
       indexes are NOT used at any given time -> 
       only ONE. hence it takes more time.
     - filter/sort by single indexes and the 
        intersection might. be very smal;l too!
     - all that given - it would prolly be the second 
       best solution
3. graph
    - no idea how.
4. columnar
    - sharding key/row id? hashtag.
    - column1: timestamp, tweet. pagination 
       sorted - no processing - only appending.
    - popular tweets? popular and recent are 
       relatively independent
    - so another column family of popular tweets?
    - prefix match? value = number of likes, but 
       fixed size. so its sorted on timestamp and indexed on value...
    - column2: timestamp(use-less), num_like+message+id
     - every 10mins: update likes for all tweets? 
     - every 60 mins: reinsert new data, purge data 
        beyond certain count - overwrite is costlier


can we store object also? in columnar, instead of simply appending like a string?

if row_id is the shard key , wont all the queries for #Diwali, go to the same shard?


how do we handle overloading in shards? how do we maximise the throughput for each shard?


SCENARIO 2:

Live match scores in a tournament? : meaning multiple teams in a tournament.

columnar? WHY?
key-value? WHY?

can use? 
1. document
2. sql

definitely not? graph.  
-------------------------------------------------------
factors?

- size of data per match?  very small
------- key, value or columnar ----------------

- structure is flexible or fixed? fixed
--------- sql or key value or columnar -------------

- data write (create/update) operations per second? - very low ~ 1 update/second
----- dont want whole doc to be replaced --

- data reads ~ very very high -> 1bn users per second
----------smallers possible parsing time and query time------------------------


n key value we update the entire value everytime?

does read speed have anything to do with choosing key value store?

but we can use other db as well on top of that we can use cache?

for every ball we need to update multiple key value pairs . i think it is performacne issue
-----------------------------------------------------------------

-> we dont even need to parse the value. Directly store html and display at the browser


What type of database would we use for a group chat type application?


scenario 3: Uber

context - needs to track the live location of drivers. what to use? given a cab, where is it? we are NOT doing the nearest cab query

- columnar - time based, row_id? based on location.
- key-value - keep updating latest location.

if onyl current location = k-v store with key as cab_id and value as location

ina ride - you want the route -> columnar with row_id as cab_id? NO!

trip_id would be the row_id

SCENARIO 4:  Group chat application?

1. relational: too much data
2. sharding - nosql:  what type ?

- columnar : key and value.
group_id and chat - with index on group_id: 
but also pagination perhaps.


can value in column db consist of complex data type/object or just a simple string?


difference between time series Db and col-family DB?


fun fact:

mongo db 1 doc = 16mb upper limit ; and We can embed docs in MongoDB though.

What is the mapping between posts and comments?
What is pipeline  and it is specifically used in mongo db

sharding in nosql - things like geo-based or other variable factors that we studied with sql wont happen right? like uber, zomato cases...?

if we need to get data from multiple shards, how we will query in that case using ids only ?


Can we discuss Zerodha's DB : for Stock prices time series view. Will it be like Cricket match scores KV Cache, or like Columnar for timeseries. Or is consistency requirement removes NoSQL DBs as options.

in the 1st sql session, you told that nosql db's are generally used along with sql and not independently. Could you elaborate on this? how to these db's complement each other?

stemming, lematisation, word vector, title and context etc,
-----------------------------------------
tg_vector etc
------------------------------------------
quad tree!
---------------------------------------------

as users grow - and the number of events that our system handles grow - sql runs out of juice and nosql db specially handles requiremenets.

you always start withg sql
------------------------------------------------

comments on MERN stack using only mongodb: simplicity
------------------------------------------------


can value in column db consist of complex data type/object or just a simple string? 

what does this mean? In CouchDB, database contains documents. In MongoDB, database contains collections and collection contains documents.

MongoDB offers Transaction support. Not sure if it is as consistent as RDBMS, but can NoSQL be Consistent as well?

NO.
concept of tunable consistency. mongo does have transaction support but only A and D will be taken care of.

- stale reads, weak consistency and isolation would still be a problem

- news flash: has ABSOLUTE ACID guarantees in mongo since 2018!
- prolly through locks

check out the link


Consistency in ACID != Consistency in CAP
how and why?

eard of BASE theorem in NoSQL DBs. Can you explain and compare with ACID.

whats the diff between column family vs wide column vs columnar?

in Column base DB: so the key is row-id and value is the specific name column ?


how does document DB look up query internally ? how does doc db store internally ?

can you please write and show us one sample query to fetch the data for a given key/uuid 

ooooh cool thing wrt sectors on disk and nosql

As developer it might be difficult to be comfortable with too many DBs intricacies. Can you list a minimal set of DBs, which give a developer well rounded experience of DBs. May be PostGres, Redis, MongoDB, Cassandra. Is that a decent choice? And What are your pet DBs.

you cannt hash mutable data types. hence only strings.

psql, mongo, redis, elastic and cassandra/hbase

how to store multive values of the same type

any company values the ability to learn quickly rather than know speciifc things

how it will create uuid using hash or augenerated while we are inserting data?

What will we have in the value once index has been created?

uuid v4 awesome!

how to store multiple values of the same Type/category

doesnt mongo have something called an object id for each document - is it not the index?

So as Redis is single threaded, can we still create a Connection pool of multiple clients, or we should have just 1 client due to Redis's single threadedness?

connection pool has nothing to do with number of threads

OK. SO its more of network delay for which we have pool - yes prolly

Why to use the graph deb in Knowledge Management?

given a graph: 
- find a shortcvut between two nodes
- knowledge graph.