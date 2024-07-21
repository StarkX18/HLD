https://docs.google.com/document/d/1qVUOgfGeFeB8Xh8Zay-NbCjgDUjPVlYaN7_y8S2BTtE/edit


Agenda:
1. Problem Statement
2. Write Ahead Log
3. LSM Tree
4. Bloom Filter

Recap:

On NoSQL: how we did following things - 
- sharding automatically
- manage servers etc

- not like DSA:  not clear understanding required - only overview

- class will cover 20 percent. -80 percent using hw

Problem Statement:
- SQL - well defined schema - know exact size of each row, due to defined col datatype...

- NoSQL: no such restriction - hence unlimited possibilities!

How are they getting stored on disk

How to save NoSQL on disk?

- cant do fixed blocks, atleast adjacently else overwrites might happen

- like a linked list? fragmentation as difficult to read this data as too many disk seeks! -> the costliest operations during read operations per minute! -> sequential read 6800 rpm, but seek is upwards 100ms

- same holds for SSD - even with different architecture and completely  stationary parts !

https://www.youtube.com/watch?v=5Mh3o886qpg&ab_channel=BranchEducation


update or write ops on sql -> very fast.

Index management in sql? B+ tree -> logN disk seeks as every level is stored in a different disk block!

NoSQL should be optimised for read and writes!

For any DB, especially noSQL as SQL can turn it off!

-> Write ahead log - a pure append only file - for any writes!
-> allows to replay the entire history of the database!
-> in case of power failures etc are recovered using write ahead logs


is WAL not flushed after writing into disk?
t is never refreshed?

with data? o r only logs like time and user who have updated?
Vadhiraju
To: Everyone
9:26 PM
Any purging happen in WAL?
Nikhil Agrawal
To: Everyone
9:26 PM
how to persist WAL?
Shriram Pasrija
To: Everyone
9:27 PM
so if server is corrupted and WAL is gone, nothing can be done right ?
Prasanth Yerrapragada
To: Everyone
9:27 PM
dont RDBMS do the same with transaction logs?



Use cases for WAL:
- Master slave replication
- db failures due to any reason
- purged in case of backups etc
- point of time recovery
- improve speed of writes in case of sqlite

It can also be read - you'll ofc want to start reading from the bottom!

can you show what wal looks like in actual?
Shashank Singhal,
To: Everyone
9:30 PM
thats why in sql there is large size of mdf files

are we supposed to turn it off in sql dbs?

is WAL outside the db

Problem Statement : how to optimise reads/writes for noSQL, for now, key-value!

1. most carefreee: 
- simple file.reads and writes happen with some formatting maybe...

2.  use a WAL:
- every write is simply appended to the file
- so for writes, duplicates might be stored. for reads, simply go from last to first dujring linear scan?

- writes are very fast. reads are still slow!

3. B-tree -> sorted structure will lower write speed...

4. Hashmap(in-memory) : entry to line number mapping.
- reads will be super fast,
- writes will be slower (extra time for returning from hashmap) but still very fast!

Any cons?
- too many keys? no! keep purging and updating?
- still if unique rows too many?
-- lets see

code for hashmap:

- you will still need to know how many bytes to read after the hashmap : given a particular offset - or maybe a sentinel too!
- hashmapping strings will be O(N) still :) - SWC Pro!
- still duplicates in WAL ... why even do the appending stuff?
- well. -the reason for appending in WAL is because data might have varied size...might cause overflow!
- hence never overwrite - append only.. :)

Problems remaining:

1. still duplication of data - space is super cheap, but wont work for large case

- so dont completely purge - atleast minimise

2. there could be significant time the HM is built when system reboots, Hashmap still in In-Memory - Power failure may cause loss of this hashmap
Limited to RAM

-> if power failure happens - you will have to reconstruct the hashmap. its easy and scalable

3. as mentioned earlier, size of hashmap might become too large! cant use it on disk -> how to address??


Adressing the issues:

1. duplication issue: periodic cronjob to purge oldest entries of duplicate keys
- it will also update hashmap ...

- fragmentation issue: creating new WAL - only copying the old but relevant data!
- hence the updation of hashmap is required since the offsets change!

- isnt creating new WAL costly?
-----------------------------------------------------------------
Addressing the problems
- costly operation to create new WAL, considering its a HUGE HUGE file.
- can we find duplicates efficiently?
- how to store WAL - huge file?
-----------------------------------------------------------------
- Store WAL in chunks of N MB -> based on some factor...not time based but size based since traffic can be in spurts..

- any writes that happen get stored in the latest WAL files -> and ONLY store the latest chunk in-memory

- writes are even faster but in case of power failure -> it snaps again.

- to ensure data not-lost ->  make another backup-WAL which is ALSO appended to -  BUT on the disk. Only for the LATEST WAL ! this way you create simultaneous WALs as well as address the power fasilure scenario!

Talking of the latest-WAL:
- any write speeds downside? No - since persiting will anyways require storing to disk..

- this doesnt need to be append only? means - doesnt need to have duplicates!
- but in memory WAL -> like a mem-table / hashmap -> since in-memory donk need to worry about disk seeks etc

-> thus the in-memory mem-table wont have duplicates AT ALL 
-> it will ALSO have the latest data!
-> read can return from memory - doesnt have to be fetched from memory!


but the WAL that is persisted into disc will still be append only right? only at the time of rollover, we replace the existing WAL with this in mem WAL.

but what happens to memtable's sibling on disk?

hashmap of keys with offset is still will be becoming huge as new keys are added\
After some writes - mem-table gets large!
-> dump to disk
-> new WAL like before

Any WAL chunk wont have any duplicates.
Thus - within the WAL chunk - no duplicates
But ACROSS WAL on disks -> no duplicates

Now COMPACTION of the in-disk WAL files can happen using a background process periodically

the background Compaction process and the actual read and write operations in the WAL
- happen at separate locations!

Mem-table -> in-memory will prolly be a hashmap or BBST!

- hashmap: 
 - sparse / larger space! - in-memory
 - indexing read and write is O(1)
 - dumping on disk looses hashmap property!

--- the MOST intuitive DS is NOT the correct one here - due to first and last points! -> it becomes useless given the data is gonna be huge

- BBST:
- no EXTRA space is required! - dense table
- indexing, reads and writes are O(logN)
- dump it on disk using in-order traversal -> will have sorted keys!

----------- so now even disk seeks WITHIN disk WAL files becomes easier due to binary search!---

Practical example:

- store in-memory WAL using BBST
- disk WALs are sorted
- for reads - if not found in-memory - go to disk
- in case of power failure - the latest disk-WAL is used to construct out mem-table.
-------------------------------------------------------
questions:

we dont use offset here ?

But the values are all of different sizes, so don't we need offset?

but value size is not fixed, how can we do a binary search?
-------------------------------------------------------

Addressed issues till now:

1. power failure - done
2. mem table is full - done

3. backgroumd compaction process???


Background compaction process:

1. merge files together : using mergesort algortihm!

how many files? keep merging until fileSize becomes equal to chunk size...

-> as these files keep merging together : it creates an LSM tree! since no duplicateas within the file but duplicates across them

-> the actual backup file (after merging) is called WAL file
-> the files consumed using compaction are called Sorted Streing Tasbles

-> cassandra, dynamo etc use these LSM trees and tuning the compaxction process is very important for these DBs...

Most of the functionality of NoSQL DBs is based on WAL files!

-> Problem statement: optimise the size of the hashmap?

-> Ashrith Kashyap
To: Everyone
10:56 PM
chunk sizes are declared only for the last level of the lsm tree?  NO - every level

-> compaction only reduces the duplicate keys

LSM Tree: how it looks - not very talll

MemTable (in-memory BBST) [root] -> SS tables as children -> as we go down -> size increases

Where is the hashmap?
Each SS table is sorted. 1GB size. Can we logically split into 64Kb blocks? Can you split into logical blocks?

Why 64kb? later...

See the maths -> 16k chunks assuming above split...

-> in the hashmap:  you dont store all the keys, plus, you DON't store key-value, but rather, offset -> 1GB -> 1 MB

-> this is a sparse index! of 1MB

-> for EACH of these SS tables: store a sparse index. And traverse?

-> now when a read happens - simply rely on the latest updated key for each of these indices! -> use offsets to pin point location!

-> so we do linear tables across SS tables, but binary search within them..


questions:

Are we storing SStables at each level even after compaction?

Can you please explain how duplicate keys across SS tables are handled?

we need to find which block to apply Binary search we need to binary search whole SSTable correct?

Now are reads are very fast.

But -> what if the key isnt there? Its still fast -> but can we optimise? since its currently going through all the sparse indexes!

Bloom filter - a probabilistic DS, which has two ops

- insert(key) and contains(key)

-> feature of this filter? If key is present - it will definitely return true

->but if not -> there's a HIGH probability it will return false

Why Bloom filter? Why not a simple hashmap?

As keys increase - hashmap size increases by ALOT!
Bloom filter is FIX#ED szied, no matter how many keys you feed into it :(

    How a bloom filter works...

- multiple hash functions
- if ALL bits contained within the array index -> pointed to by these hash functions are TRUE -> return true.

If ANY one of them is false - return false!

- hashing then becomes a key thing here!

https://hur.st/bloomfilter/

https://llimllib.github.io/bloomfilter-tutorial/

Good point!

but at a certain point all indexes become True because size to low...

Probability of false positive using bloom filter...
- k is no of hash functions

https://www.youtube.com/watch?v=5Mh3o886qpg&ab_channel=BranchEducation

Use cases of bloom filter:

1. Akamai, Cloudflare CDNs - 
- 70% of URLs are visited only once!
- use bloom filter tas criteria for caching

2. NoSQL DBs as discussed almost always

Deleting a key in our LSM tree + hashmap + Memtable!

- if we JUST delete from mem-table -> everytime all Sorted String Tables

- if we JUST delete from bloom filter -> not possible! bloom filter never forgets!

- another bloom filter to store deleted keys? what if same key re-inserted? Both wont forget. So is that the key deleted or re-inserted?

So how to delete?
- tombstone value - like a sentinel!

- write the value as tombstone value for the particular key. If value = tombstone: its deleted!

- do we delte from the SS tables? NOT required! Because when SS Tables merged - only the latest data would be writteon onto it! Which would be the tombstone value..

what benefit do we get from compacting SSTables other than saving on storage?
tombstone is basically saying RIP key

will it be eventually cleared out with some cron job? NO!

why is tombstone a good solution? 

- bloom filter always says key present
- go to SS Tables -> binary search -> tombstone..

if read request comes and key not present in mem table, then it searches in sparsh index leading to lot of read from sstable , then read can be expensive ? 

when we are doing compaction that time may be to log files have same data so which we have to pick how we know which one is the latest one is it based on the names?
Why SS Tables compaction?
- huge size + speed optimisation...

if read request comes and key not present in mem table, then it searches in sparsh index leading to lot of read from sstable , then read can be expensive ? 

sparse table - gives one lpocation of 64kb block = 1 track size on your disk

binary search leads to logN disk seeks -> ans number of SS Tables is logN !

if you fetch latest data in fact - which is in-memory...

this performance actually becomes better than the B+ trees performance in DBs!

Binary search takes 1 disk seek! However the performance is still better than the B+ trees performance!


when we are doing compaction that time may be two SS Tables have same data so which we have to pick how we know which one is the latest one is it based on the names?

yes basis of names or timestamp! 

WAL is just used for recovery from failures now...

What is the difference between SS tables and WAL?

What is the difference between SS tables and WAL?
Summary:

1. large WAL file - writes fast, reads slow
2. hashmap - to save offsets
     - too large due to sparse - wont fit in- 
        memory
     - dupicate keys?
3. We now want to remove duplicates and optimise hashmap

- split WAL into chunks
- most recent is in-memory
- store mem-table in BBST
- store it after a fixed size is reached - into disk -> call SS Table
- using in-order traversal: it becomes sorted string


WAL is append only + a more generic for all DBs - 
SS Tables are dumps made by in-memory mem-table

how the Changing size is affecting bplus tree aproach

Each node in B+ tree is 1 block -> 1 track of 64kb

any block read = exaxctly 1 disk seek

- if data is varying: becomes different


how search happens in sparse index as it has only first key? is it binary search or is it hash based to find offset of block on disk?


Pragy, do the sparse indexes associated with each SS table reside in memory?


is the WAL chunked only for NoSql or this happens even for RDBMS?

Does happen for RDBMS but for diffferent reason...

- SQLite for example - will write to WAL -> then periodically and async do the actually write - the expensive part in B+ tree

- but the chunking part isnt done in RDBMS - because they have fixed schema and can always update existing entry

How do we know which sparse index is associated with which SS table? Do we maintain a mapping in memory for that?

can you show WAL config in any data base?

Different DBs use completely different WAL file!

Everytime creating a WAL file is expensive?


What are the NoSQL databases that use this kind of Memtable/SSTable mechanism? Could you please share some names? Thanks.

BigTable by Google, cassandra, dynamo etc

LSM trees are used in data stores such as Apache AsterixDB, Bigtable, HBase, LevelDB, Apache Accumulo, SQLite4, Tarantool, RocksDB, WiredTiger, Apache Cassandra, InfluxDB, YugabyteDB, and ScyllaDB.