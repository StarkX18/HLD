Agenda: FIRST official case study - like a JOB on hand

1. Problem-statement
---------------------------------------------------
2. min viable product
- min core features, not fancy things
---------------------------------------------------
3. scale estimation 
- queries/sec or 
- storage requirements - sharding required or not?
- read or write heavy?
---------------------------------------------------
4. design goals
- CAP
- is consistency important? can we settle for eventual conisistency? can we deal with data loss?
- latency requirements? PACEL theorem.
- Do we also consider what kind of system it is: Data intensive or Compute intensive? 
- how does it effect our design strategy?
- examples of compute intensive? online judge, blockchain mining, chatgpt -> rare apps!!!!
----------------------------------------------------
5. APIs:
- how is the external world using it?
----------------------------------------------------
6. System Design
----------------------------------------------------
unlike dsa - discussion can go in any direction
----------------------------------------------------
varaprasad: resolution dstrategies for failure of replication is not discussed like vecto clco

Do we also consider what kind of system it is: Data intensive or Compute intensive?

Problem Statement:
- not like your typical phone - which is a very small scale thing.

- a large scale - google scale!

Minimal Viable Product - your inputs?

- how many suggestions? 5 (say)

- where's the data coming from? what's the pool of suggestions?
    - popular searches or history

- user personalisation important or same set of results for all users... NOT important for MVP (say)

- how large can the search query can be? infinite - try your best! - based on algos? should we show irrelevant results? like some websites? - only relevant results...

- is it the contains or begins with query? typeahead does prefix matching 

- how do we deal with spelling errors? is fuzzy search allowed?

- how many characters be typed to start typeahead..?

- debounced or per character updates?
    - do we show results at every 
       keystroke or after a little delay?

- algorithmic question: how to rank the results?

tip:
try to think of the SIMPLEST features!!

- things like user-personalisation, etc isn't for MVP
- your job is to build the product - not brainstorm - you are here to build and get shit done!


it must have a feedback mechanism - the actual query that user clicks is the relevant one - and shoudl be ranked higher

- search queries are different from type-ahead queries

mostly for this kind of work we use elastic search? NO

MVP discusses what we are building ->  what the user will see and use?

- Things the user wont care about -> 
   should NOT be discussed here

Design goals discuss how you want the system to behave -> the product is different from the design of the product

Scale estimation: the context is google.
Pure scale... :)

- Volume of queries: 10bn search queries per day ~ 10^10 / 24*3600 ~ 10^5 searches per second

- but thats for search! fuck! type ahead will have even more queries...

- assuming 10 typeahead queries per search query ~ 10mn queries/sec

- if the numbers are wayyy too off -> interrupt

More scale estimation..

1. do we need sharding? scale of data?
- writes per second? reads per second?

but before that!
0. what are you gonna store??? 
- suggestion? word:rank
 ----> when rank changes all of the ranks for 10bn words need to be updated?

0A. what are you storing and for what purpose?
- words/ search-queries 
- number of hits
- timestamp? - Pragy's fix: not required - someone somewhere is searching for everything ... all queries are recents! lets tackle recency later...

good question:

you said that google search will keeping track of most popular searches. why do we need to incorporate this as a write query in our system?

10bn searches per day ~ 100bn typeahead searches per day
- so maybe 3 months of data?
- NOOOOO - we dont want only 3 months - we want all data??
-------------------------------------------------------
- how much data? 100bn typeahead queries per day * (32 characters per query + 8 bytes for count)= 100 bn bytes per day = 100bn * 365 * 10yrs = 1000 * 365 bn bytes ~ 5,00,000 bn bytes = 5*10^15 bytes 
-------------------------------------------------------
But this is not the relevant stats here!!!

we want the NUMBER of UNIQUE queries per day - ie never searched before!!!!

- most of these are garbled too though!!
-------------------------------------------------------
thus - this number becomes a lot smaller!
-------------------------------------------------------
we arrived at 1.5 * 365 * 10^14 ~ 10^15 ~ 1petabyte or 1000 tb of data
-------------------------------------------------------

Now tell us? will we need sharding?
--------------------------------------------------------
-> some systems can now handle 1PB of data! tinkers with servers
- youtube linus tech tips
-------------------------------------------------------
even though the system might store data - but not the requests.
-------------------------------------------------------
read heavy or write heavy system?
- 80:20 rule not applicable - instead for every search query there are 7 typeahead queries

- every search will increment the count - so write happens
- but also - every search has ~10x more typeahead queries

- but still this system wont exactly be read or write heavy - its both
-------------------------------------------------------

Design goals:
------------------------------------------------------
1. do we need consistency at all? can we deal with data loss? 
-------------------------------------------------------
2. consistency vs latency
-------------------------------------------------------
3. consistency vs availability?
-------------------------------------------------------

availability..is preferred over consistency
latency too...is preferred over consistency

so latency should be insanely low as you are competing with the typing speed of the user!!
------------------------------------------------------ 


APIs: how is the external world - outside to the current service / end user use my service?

which type - REST/SOAP/gRPC doesnt matter
------------------------------------------------------
1. users/other services at google: 

- 2 main APIs: 
------------------------------------------------------ 
- suggest?partialsearchquery=querystring

- updateFrequency: google's search will use this - async
-------------------------------------------------------

Actual design: Typeahead based on above premises
------------------------------------------------------
2 APIs - writes < read
so tackling write first
=================================
Search services sends update - we update the frequency...but where?

what kind of db are we using? not the most relevant question.. 

how will you store the data?

1. relational? why/why not? sharding becomes difficult..

2. docdb?

3. columnar?

4. graph?

5. k-v?
------------------------------------------------------

We went the wrong path... we went write first but the minimal write first data-store might not be able to cater to the readqueries 10x more....

example for k-v store - write works well but not read..

Starting with read this time...
------------------------------------------------------
read query :

-> input = prefix
-> output = 5 typeahead results
------------------------------------------------------
shards should contains all the prefixes! so the basis of sharding becomes clear..
-------------------------------------------------------
Most intuitive approach: Trie based data store/ hashmap/ key-value
-------------------------------------------------------

Trie based approach:
------------------------------------------------------
- construct a trie out of all the search queries
- children[36], isWord, #hits

-> increase hits for each word that comes along
------------------------------------------------------
suggest API:

- again reads can be optimised by using extra DS within the nodes.
- but. write opes become heavy - as every rank update needs lots of reoptimisation

------------------------------------------------------

-> 2 hashmaps approach -> top 5 results for each prefix and prefix to count matching!
------------------------------------------------------
Tries are for space optimisation : not for low latency or speed!
------------------------------------------------------

So we use 2 hashmaps: final results!
but updating top 5 results for every update is expensive...
------------------------------------------------------
- the key-value store is automatically
sharded based on key.
- so reads are optimised...
------------------------------------------------------

Optimising the writes:

- every write is updating multiple prefices. : infact it might even exceed the number of reads suddenly
-----------------------------------------------------
- also - you cant perform multiple writes in parallel 
- and it locks the rows - so simultaneous reads are not allowed during this time

- read locks might not be needed for our evental consistent systen but write locks are still needed.

- coz we also want to take care of race conditions
------------------------------------------------------

we can have master-slave? no but the problem is not solved. we cant handle so many requests !!

messaging queues are shock aqbsorv\bers, not continuous machines

1. sampling the data.
2. threshold approach - after every threshold seconds/ threshold writes / batch write from cache to db - can even do tihs via key too - like batching key-wise...
3. redis i s good

accomodating recency factor?
time decay for hits...like fission

how will it affect our decision exactly?
apart from hardware requirements...?


in top 5 prefix still the updates is same as trie? but here we are updating it on 1000th

in case of sampling approach where we are doing sampling? we are updating exact count , right?

no of hits is different for "Diwali", "Diw" and "Diwali 2023" right? if people are searching like these