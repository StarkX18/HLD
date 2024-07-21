No SQL?

Say no to sql? no 
not structured query language? no

Not Only SQL - yes
Because we will prolly not use ONLY sql

- however core will/shoudl have sql prolly

Types of NoSQL:

key-value
document
column

Use cases of NoSQL:

- Data is massive
- SQL doesnt satisfy your needs

the good parts of relational database:
PostgreSQL: incredibly powerful, tons of features etc

- tables: 
      - cols: define schema
      - rows: actual data
- tables form relationships using FK constraints
- hence the possibility of complex queries
- normalisation is possible and easy: ie removing redundancy
    - hence helping in data consistency across tables!


anomaly without normalisation
- updation everywhere across tables
- deletion properly acroosss tables
-  one more

what was it? creating extra tables and relationships between them - using mapping table etc


example: pin code and address and city and state in the context of normalisation

A - atomicity - all or none - using transactions
C - consistency - invariance is maintained 
I - isolation - transactions should not interfere - no dirty reads - which means a read changes due to another transaction while one is going on!
D - durability - completed transactions are persisted in case of system crashes - due to external factors 


however - transactions are NOT guaranteed to be INDEPENDENT of each other.

Read the concept of serialisable.

Also - these things are not provided by external oarties but by the database itself!


cap consistency means all different servers can access same data and acid C says before and after any transcation data is consistent right


- joins and complex queries...
- strongly defined / rigid schema
- each column has a datatype and each row has a fixed set of columns

shortcomings of non-relational databases:

- rigid schema: if data is highly flexible, can be a problem

Shortcoming of non relational DB:
1. rigidity in schema for lots use cases:
- example : e-commerce will have lots of unstructured data as all products will have a different set of attributes.

- even in this sparse db : we dont know whether null means not present or not possible or what else!

- Kaushik DAS
To: Everyone
9:59 PM
columnar databases will not have this problem of null values right

- what we can do - create a table for each product type/category.

- SQL is optimised for large number of rows! but NOT optimised for large number of tables.

If there is alot of data - single machine can not handle the load!
-> num_slaves proportional read throughput
-> not to write throughputs

-> only solution to increase write throughput - sharding nullifies SQL advantages!

- wityhin shartds all ghood, but problems come with across shards.
- cross shards - joins: difficult
                              - relations, undsupporting etc

- ACID guarantees of transactions - go out! you have to manually support DBs!

- no official support for sharding - postgrers, sqlite!

fb news feed: feed of sharding supported by cache/another db! - with limmited timespam!

- if you go beyond timespan: query will be across shards:

- most systems dont allow!
- fb/insta reminds , your'e alll caught up, if we scroll very much.
- google doesnt allopw you to go beyond 50th page! - captcha and stuff!


Hence sharding which is significant, isnt supported!

- NoSQL DBs into the play!
types:  key-value, documents, columnar, time-series, graph

- surrealDB*** AWESOME, cassandra, neo4j

choosing a good sharding mechanism!

- chat/messenger app: user-id(default and good), timestamp(poor for this use-case! - if you wanna fetch messages)


context: given a conversation: fetch recent messages - or all messages!

you cant decide based on pure ..... need context!

Would that simplify the queries if we're supposed to model DB around one-to-one messages and one-to-many messages?


if you choose shard key as 
- tx id or rx id: creates problems

the difference between db modelling -  

conversation view vs all notifications regarding unread messages


when we know the possible queries, can we have multiple shards?

need one more table to store whether conversation is read or not
storing using timestamp - or geolocation

solution:

shard data by user_id
but any conversations would be stored both in senders and receivers' shard

so as a whole our database has duplicates for each message which is against normalisation right?


writes can hit to 2 shards for 1:1 chat, but if its group chat writes can hit multiple shards right, how can we handle that?


another example:

- list all accounts,
- transaction history
- balance
- make_transaction(sender, receiver, amount)


best sharding key?

- user id or account id -> account id is too granular! doesnt store user details

- cons of other keys

a. location? shitty for many reasons - examplke if user moves - copy transactions to new location
- only make transaction makes difficult - user and reciever in different shards
- some cities are more active - so not uniformly distributed!

b. user_id? all queries except make_transaction only needs 1 shard


ride sharing system - 
context: get nearest available cabs


possibilities:

driver_id: nearby drivers might. belong to different shards
city_id: find nearby cabs.

even ontercity - only 1 shard

irctc:
context: tatkal ticket booking - userr-zid, from, to, date, class

i answered - location
answer - train id: 27k bookings per minute ~ 200 bookings per minute

what if different db for tatkal?

date of booking? very promising: double booking handling is guaranteed!
whats the problem?
shards will be OVERLOADED! on traffic up!


why train id? 
no double booking.

you CANT have ACID guarantees ACROSS shards. Its guaranteed within same shard


can we do some practical here , all things are very theoritical , how we shard ,
can u pls show !!!
Rahul Mishra
To: Everyone
11:11 PM
same train can travel on multiple days so days shard key will increase difficulty?


if the intewrviewer has to point out the mistakes you have already lost

where will we maintain source, destination -> train info?
In other sense, if I search source to destination , how will we fetch all the trains for the combination without going to all shards


choosing sharding key: basis of pros and cons

1. load distribution
2. most frequest operations should be efficient, even if at the cost of least efficient ones
3. most frequent writes -> least number of shards need to be updated -> rollbacks, cancellation, consistency
4. data duplicacy(NOT redundancy) is minimised, if not perfectly avoided


positive only caching 
vs 
negative caching - store even negative result like AoT on netflix

do you need to shard user_id in irctc?
10 mn users ~ sql can handle

users: id, name, users
stations: id, name, platform
trains:
destination etc...

read queries for each of these can be handled

bookings: id, user_id, train_id, src, 
if read queries are largew-  2e will do replicating, caching etc

bulk queries for writing!

bookings - train_id

You can answer this in doubt session: writes can hit to 2 shards for 1:1 chat, but if its group chat writes can hit multiple shards right, how can we handle that?


for 1:1 chats - its ok for sharding, how to handle group chats?

- like slack, whatsapp, discord etc?
- do we copy to 1mn shards? ofc not!

- here shard by group id...
- user-chats, group-chats are seperate db with different shard keys

unread messages in slack vs whatsapp! - 

-> unread stops for whatsapp when you leave, not for slack! -> so [prolly channel id sharding?

-> notifications are NOT stored in db - but websockets - kafka queue etc

-> webhooks?websockets?
----------------------------------------------------------------

-> device storage on whatsapp..


where will we maintain source, destination -> train info?
In other sense, if I search source to destination , how will we fetch all the trains for the combination without going to all shards

should we do sharding for both sql nd non-sql , 

or only with no-sql ?

important!
- sql - sharding necessary
- no sql - automatically happens! you wont even have to specify! out of the box

can we discuss a case study on banks data storage & scale in end?

explained why banks are slow - strong consistency is desirable.

if transaction fails/ not written to all servers - its rolled back after lots of time

CRDTs - git
and 
blockchains

ATM: transaction passed for you - but failed for bank - problem!

will we have all the records/tables wrt the shard key in a single shard

currently implementing notification task in my office task, what if the user is not online? when web sockets dont work right? then notifications will have to be persisted in a db? or could you explain the message queue solution in a little more detail 

push notification...
shock event of a pew die pie: spring/queue -> db consumes -> send to users
-  indexed db, SWR, client side cache, tanstack etc
- redis, graphQL, etc for this case

**** noice question!!!


Instead of pushing Notifications to subscribers, cannot subscribers check if  their subscribed channel has uploaded any new video and generate a notification.

pull instead of push mechanisms for notifications...
- number of events increase drastically
see how

can we do some practical here , all things are very theoritical , how we shard ,
can u pls show !!!

not clearon ictc and uber things of sharding?

Does data localization (US data stays in US, Europe in Europe) also a case for sharding 


yes. like zomato. shards based on locality.
otherwise on use-case

Hey Pragy, what should we do if there is a new requirement that is difficult to achieve by the shard_key that was used earlier and was optimal for earlier requirements? How to deal with this situation?

bad scenario:
1. anticipate: developing solutions is difficult and costly 
2. dealing with it - salvage plus build some new tech


Can you show any practical examples where someone is supposed to combine sharding and replication to favour availablity(CAP) of db ?

cube of data size vs conbsistency

how to choose btw having replica and sharding ?

not clear on irctc.. why not userid, why train id.. how userid can cause double booking and how trainid cannot?

How to ensure consistency in case of sharding?

pragya in notes whaterever q&N we  discuss it will updated if u use in same doucmnet.. so when we are taking  print then we lost some data

cache replicas will still have to follow master slave?

why can we consider user geo location as sharding key for uber ride booking

how to pre calculate number of tables? in case of product types it is easy, but what if we don't know about number of tables, then is it advailsable to use only NOSQL?