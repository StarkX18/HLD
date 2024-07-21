Agenda:
1. Full text Search
2. Elastic Search
3. Sharding

role of zookeeper is just config management ?
3 points...see !

Zookeeper = config management + load balancer + health check

Problem Statement:

- you have a large volume of docs
- you want to search these docs for specific words
- text queries for whole words, not partials or regex

Use cases of our problem statement?

1. logs - tracing, analytics, security, monitoring etc
2. ecommerce 
     - searching relevant products title , descriptions etc
     - relevant services
3. search engines (on a completely different class and scale)
4. social media profiles - linkedin / twitter
5. messaging chats search - slack, MS Teams
6. scaler problem search
7. drive/gmail / email search
---------------------------------------------------
1. Note: we are talking about EXACT searches, not grep like regex search

- this becomes massively more difficult
---------------------------------------------------
grep = g/re/p
global reg exp print
---------------------------------------------------

Amazon reviews / logs: 

due to large number of docs: noSql-doc db!

is sql a good option for our problem statement?
no, it scans each row !

even adding index wont help!

Even inside a noSQL DB - cant search full words without scanning through all docs

any index/trie will only be useful for prefix search/startwith queries

concept of inverted index: similar to hashmap

- split the document by words
- with word as key, store metadata like doc number, offset etc for each word

why call it inverted index? intuitive

inverted index is similar to glossary in a book, index in a book is index

problems with this approach?
- the list of docs can be very high

Elastic search/Apache Lucene/ Solr etc all follow a flow.

1. remove the STOP words :{the, of, is, was...etc}, special chars, etc depending on usecase

2. Stemming/Lemmatisation: 
stemmin
=> common postfixes are ridden of {ing, s, tion, ion} etc => a very fast process
but not robust

lemmatisation
=> uses the context to stem - using NLP/ML etc

3. Other cleaning: abusive/sensitive words, racist,terror etc

4. inverted index is built 

=> When a query is sent,
the same steps ie stop words, lemmatisation etc and then sent forward

=>  eg: the woman with the white words = woman white dress

now how do you search the whole term?
-> woman on i, white on i+1, dress on i+2

this is left to service layer - the implementation

this index can be very very large - and the db storing the docs too

- high load, SPoF, run out of space

what are the system requirements?
- consistency? aavailability? partition tolerance?

Also consistency in what context? ACID or CAP? Note they are different!

what will immediate cosistency look like?
no doc that matches the query is left out

if doc is updated, it should be immeidately updated

eventual consistency? search index might return stale results


Here we feel like eventual consistency is good enough

how to shard?
- based on document id - can be review, log, product or post id - what to search basically
- based on word


what is practically used? doc id in lucene etc

why choose doc id?

- write query? doc id -> shard -> re index doc content

- search -> since shard is not based on word -> every shard will be searched -> return list of docs that match

- collect all docs and return

Note: we are not looking at how multiple word queries will return

Note: each and every shard maintains its own search index which are independent from each other!

Note2: in each shard, we are storing the inverted index, NOT the docs itself!

we are ofc storign SOME data od doc also, for update queries - well see later

when shard returns - it only returns doc ids

If we are storing the inverted index then why do need to shard by document ID Inverted index should have been chosen as shard key right?

we will see later

Is latency increased in search operations?

No, shards can process the queries parallely due to independent indices!

But how does pagination, sort by timestamp etc work like in kibana?

Map reduce! - the technique by which ES collects all the results

Map Reduce:
no for loops etc, only functions

map fxn - single item
reduce - two params

much used in distributed programming

MAP REDUCE in ES

map(fxn, items) -> send the queries to EACH shard which are the items

reduce(items, accumulator, fxn) -> reduce(shards, [doc ids], concatenate)

cons of doc id sharding
- search goes to all shards

now our system is already read heavy!

pros:
- latency is HIGHLY predictable, which is NOT the case with word based sharding! - due to random docs, we KNOW each shards' time

- high vol in requests might not be handled

- no need to do intersection, only unions
- if a shard goes down, the other shards still return, so availability is good

Use cases

Open source solutions: lucene, solr, open search

Proprietary : Azure, algolism?, ES, SAP HANA etc

whats the problem with word based sharding?

- unexpected latencies and sizes per shard, depending on word frequencies

- intersection will be required - for multi-word queries

- writes will happen to multiple shards - as each doc will contain many words - but not a problemsince system is read-heavy

- search queries will be faster though, as each word query will be of a few words and only those shards will be hit

- also, the output will be much smaller than output from each shard - wastage of resources

- if a shard goes down, we cant omit the results! since it will corresond to a particular word

- hence the overall performance is extremenly unpredictable

What happens if a shard contains many popular word.....will get overloaded

Zipf's law: the ith most used word occurs 1/i tumes

so this is a realistic case because the most frequent word will be overloading that shard ut cant do anything about it

Robustness and Replication:

ES has fixed num shards and replication factor - which cant be changed at a later time!

We have to build the entire solution again.

If N nodes - M shards, R replication factor => M*R items => each server stores MR/N items

Master and replica are never stored on same machine for obvious reason - i machine = 1 copy of a shard

eventual consistency + master slave replication

- misspelling/synonymns etc are done in stemming

Now, All docs that match a certain query are now found

- what else is required? ranking!

some stdd rankings already present

- TF-IDF - term freq (tf) and inverse doc 
                     freq  (1/df per word)
- title > description etc for ranking

Note - IDF helps for multi word queries

At the end, ES is a NoSQL product

https://www.quora.com/How-is-a-leader-elected-in-Apache-ZooKeeper

https://apache.googlesource.com/zookeeper/+/3d2e0d91fb2f266b32da889da53aa9a0e59e94b2/src/java/main/org/apache/zookeeper/server/quorum/FastLeaderElection.java

https://slack.engineering/search-at-slack/

https://sematext.com/blog/elasticsearch-shard-placement-control/

Generally in master slave replication, one master and one slave are each a different machine.

But in ES, only master slave replication is of shards being on different machines

common spelling mistakes - correct them using hashmap

- NFA : automata theory state machine
expected state - non deterministic finite automata

ES stores the documents in disk or in memory? And the index is in mem?

ES doesnt store docs, only word indexes and metadata
- index is also stored in disk but cached in memory

explained above

map(fxn, [items]) - apply this fxn on each item

reduce(fxn, [items], accumulator) - combines result

no need to store ALL the data in memory, just pair at a time

example: google : sort in lexicographic order given file size are in 50 PBs - 10^15

- imagine 10k machines - MAP will take sorted data from EACH machine and then REDUCE - using merge fxn

if we are sharding based on word , then we have millions of shards?wastage of space?

sharding key doesnt determine num shards!

we have documentid as the shard key and in every shard we have inverted index based storage.. can you please give an example of data model ? understanding is absolutely correct!


How to define SLA - availability, resilience of any system?

Do we have to calculate NFR (availability, resilience and partition tolerance) of any system?

For read requests from slaves DB, how to fetch all slave api’s to distribute the request in round robin?
