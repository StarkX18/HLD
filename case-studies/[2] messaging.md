Agenda:
1. Problem Statement
2. MVP
3. Scale estimation
4. Design Goals
5. APIs
6. System Design

Tips:
1. MVP : discuss core features - no fancy feature suggestion spree

2. scale estimation: storage requirements, read/write heavy, query rate, sharding?

3. design goals: CAP, PACEL, Data loss tolerance, latency requirements

4. API: how is system used by external world..

5. System design: put it together

Types of messaging apps:

1. facebook messenger
2. whatsapp
3. telegram
-------------------------------
complicated ones
4. discord
5. slack
-------------------------------

what we tackle: fb messenger

MVP: ball in your court - v0

1* sign-in / sign-up - v0

2. search anyone? contacts/friends? - NO!
- no cut to the core: sender can send message to receiver - v0

3. attachments? no - v1

4. sent / read receipts? again fancy :P - v1

5. dont confuse fb with messenger
- no user profiles / friends / DPs etc...

6. group conversations - v2

7* recent conversations

8* message history..

9. cloud or local storage?

10* what is the difference between our app and an email? real time chats!!!


basayya kulkarni
To: Everyone
9:21 PM
I am able to delete message
SAHIL NASIR
To: Everyone
9:21 PM
chat history
Bittu Kumar
To: Everyone
9:21 PM
admin of group


Eshan Akash
To: Everyone
9:22 PM
seen and unseen messages
Rupesh kumar
To: Everyone
9:22 PM
post on timeline
Rishabh Verma
To: Everyone
9:23 PM
profile picture
Bhushan Vasant
To: Everyone
9:23 PM
if receiver is not available store the msg?


Satyendra Singh
To: Everyone
9:20 PM
group message, reaction on messags
biswajit
To: Everyone
9:20 PM
should be able to retrieve recent past conversations when a user chooses to send a message to a receiver
Raj Sekhar
To: Everyone
9:20 PM
notification on new message - v0 ?
ABHAY JOHRI
To: Everyone
9:21 PM
backup of chat messages?
Kuldeep Siddabathuni
To: Everyone
9:21 PM
Notification
basayya kulkarni
To: Everyone
9:21 PM
I am able to delete message


causality is important - order of messages is imp (qn vs reply) - we should discuss here or design goals?

sort unread messgaes on top? so that messages will not be missed.

scale estimation:

- messages per second: its huge. simply simply huge
   can you estimate?
   - number of fb users? 1bn monthly 
      active users @ fb
   - each user sends 10 msgs/day ~ 10bn 
      messages/day : anecdote from 
       anshuman 
    - now real life: 70bn messages/day

- per second? assuming 70bn? 1mn messages per second

- estimate yourself / ask the interviewer

storage requirements:

1. size of 1 message ~ 20 words / 100 chars/bytes

2. metadata for each message: timestamp = 8B, receiver and sender = 16B, attachments path = 50B\\

size/message = 200B

thus daily storage requirements?
200B * 70bn messages = 14000 Bn Bytes = 14 TB/day

- say we store 20yrs worth of messages?

=> 20 * 265 * 14TB = 100k TB = 100PB

do we need sharding? yes until you can store 100PB on one machine...

Read heavy or write heavy?
- write heavy? isnt the other person simultaneously reading it?
- so both!
- by our above stats!

so complex design.

- read heavy systens? caching
- write heavy? 
- sampling, if consistency can be 
   compromised
- batching too - so write requests becomes lesser

in type-ahead, we started off with the scales balanced but dipped them to read heavy using above tactics!

In this case, can we do sampling? NO

Design problems?

1. availability vs consistency? 
- both seems REQUIRED!
eventual consistency?! or immediate??! 

analyse the cases...

- sender sees message sent, but the receiver hasnt received yet.

- messages are dropped! like iMessage - later inserted in order
-------------------------------------------------------

its a tricky bet to make! - but consistency is more important! 

so when you not available - you tell clients transparently
-------------------------------------------------------

but immediate consistency means all the replicas should have the latest data before acknowledgment is sent

thus if a message is shiwn as sent - it should be shown as sent!

-------------------------------------------------------

HLD: Desigining Data Intensive Applications - most fantastic book

sync of clocks:
https://github.com/jeffrey-xiao/papers/blob/master/textbooks/designing-data-intensive-applications.pdf

https://www.youtube.com/playlist?list=PLeKd45zvjcDFUEv_ohr_HdUFe97RItdiB

Consistency vs Latency: now we have taken a stand. always prefer consistency.

Data loss? NO

latency requirements? higher than typeahead, but still very low. say less than 5 seconds

if we have choice - we will always choose consistency.

but we can reduce latecny usinf h/w

APIs? how will external world use it?
Base on user journeys....

0. /recentMessages(userID, pageOffset, limit)

1. /getMessages(conversationID(user/groupID), 
       offset, limit)

2. /sendMessage(converationId(senderID/ receiverID) , message)

3. /getConversation()?
-------------------------------------------------------------
discarding: duty of some other app:
5. /signIn
6. /signUp
-------------------------------------------------------------

Also - why are we sending senderID? isn't our application using it already from within the codebase?

- if you in-charge of this server -> you will probably be using an auth and app server - which will be decosding th JWT etc...

-> hence its required for external use!

done till here...

read for idempotence

POST OUT are, GTE isnt

Idempotence: what kicks in for network partition

- if message is not sent first time, the second time / further retries -> might create new message! - will create anoither id if some id is attacged to message!

- we want same message to be sent - not new messages

- mathematically:  f(x) = f(f(x))

- to promise idempotence: attach a uuid - if uuid already present - wont be stored once again!

-> other techniques and why they fail - 
- Hash of message to check if signature is same or not ?
- msdId = conversationId+incrementNumber
- timestamp based
- by checking hash based on ( Timestamp+ message)

uuid? uuid-v4, guid etc - 
when this uuid is created at he time of message ?? yes

everytime validating against db could increase latency - yes it might but currently we anit discussing db or cache..

idempotence: POST vs PUT?

This issue happened with uber eats using paytm for payment

you said GET method is not idempotent

@Shriram, because in post/put call if the info is already present it will not create a new record, where as GET will fetch all the info.

sharding key...

option1 - user_id: 
all conversations of a user must be present on one shard.
- fetching recent conversations is easy
- getMessages: 

Sharding key? its tricky as one sender and one receiver

- its like email only - you can keep user id 

- but then either 
- you want to keep data in both users' shards / all users in same group

- or you can simply fetch from different shards
-----------------------------------------------------------------

exploring both approaches:
-  one side only
   - getConversations : easy
   - getMessages: easy too
   - sendMessage: hard
        - as storing in two shards
        - in case of groups: even more writes!
        - called fan-out write
        - consistency also becomes a 
           problem
----------------------------------------------------------------
question: shouldn't our architecture be extensible? YES!
----------------------------------------------------------------

quesrtions:

1.can we consider a group as user only , group conversation does not go to each user? stupid idea
- make another db for group?

2. i think the sharding key can be composite key of sender and receiver

Second approach:

conversation-id as shard key?
1. getConversations() 

- could be difficutt since each conversation might be different

- or we can store user-to-conversation data
   - latest message/snippet
   - other metadata
   - this is sharded on user_id
   - has to be sorted on recency too - for 
      recent conversations

but populating this db will be hard right? for that we will need to hit diff shards

each user will have its own copy of all conversations ?

but conversation history should be independent to all user. if one user want to delete msg it will be gone for both/all right?

- just hit that shard and getMessages from conversation
-----------------------------------------------------------------

-getMessages is simple
- sendMessage also becomes simple: we write to same shard

- but the secondary db creates issue: whenever a message gets sent, it is alsos sorted by recency too (as discussed earlier...for recency API)

- you have to go to secondary db, for each user who is part of conversartion - and mark the convo as most recent!
-----------------------------------------------------------------


fb vs slack:

-> choose between the 2 approaches: fb messenger will more likely choose the first approach
(user-id based sharding)

- slack on the other hand, will prolly use the conversation-id approach.
- it never show recent messages!! - coz it doesnt have the secondary db in SORTED order!!

-----------------------------------------------------------------

- slack started as a gaming company and couldnt  pull it off!
-----------------------------------------------------------------


cant we use columner db as secondary db which store based on timestamp for maitaing the sorted order.

- yes, but the issue will staill remain.
- also, what all would you use as rpw id and col id

maintaining the consistency...

- sendMessage(): write to sender as well as receiver shards.

what to do in case of failure?


we need something like 2PC? yes
---------------------------------------------
maintaining consistency: while sending message

SendMessage consistency:

- case1: first write to sender shard then receiver 
   shard
   - fails at sender: return failure response
   - succeeds at sender, and fails at receiver:
      - option1: 
          - immediate retries -> retry later -> 
             rollback and return failure?
          - eventual consistency
          - also - it will cause very high latency in case of failure!
          - but rollback can also be error prone because the sender may be on a different shard.
          - for consistency: we will acquire a 
             lock and do stuff - so app is unavailable for some time!

      - option2:
          - first write in receiver's shard.
          - if fails? keep retrying with exponential intervals
          - until both are written to - sender only seer - sending message. receiver might eventually get it.
           - a much better approach than previous
           - as long as message isnt sent: and if browser has no cache, sender wont see his own message!
           - but easy to work around since cache will be present...app-cache
-----------------------------------------------------
what if reciver reads the message before seneder recives success? better than prev..

if sender message has to be rolledback in case of failure but receiver has reveived?

this system also eventual consistence how it is diffrent from first case? it have low latency

choice of db?
since both read and write -> massive sharding -> no sql - since sql dies at around bn users

- no good system optimised for bioth reads and writes! 

- convert this system to either read or write heavy!

- howver previous tactics of write-aheaf wont work since ot was eventually consistent -> infact - ours is immediately consistent!

now;
- write heavy - is important since immediate consustency is REALLY important

- so make it writw-heavy! removing read heavy...
- by ensuring most reads never go to db!
- heavy caching: huge cache plus 
- even cache would be sharded - it will contain almost ALL the data from db

- for consistency of cache with db - write-through cache... not a write back since it can loose data!

- it ALSO has to have high concurrency!
- make sure you handle high concurerency - user might be sending from multiple devices and very quickly...

-> acquire locks - on both cache and db level...
-> which implies NOT a global cache! but a local one...so our app-servers become stateful
-> so now LB will use consistent hashing based on user-id

-> acquire a lock on user-id

Pros and cons of our approach:

1. scales horizontally: more users = more servers
2. race consitions are gracefully hadled
----------------------------------------------------------

- if app server goes down: till the time user is assigned a new server - cold start happens!

- service would be unavailable for some time: to this PARTICULAR user mitigate with replication...

- in that time - user can believe internet is bad :)

now a write heavy db:

- column family due to recency factor
- hbase/cassandra - hbase is defacto - HUGE write capacity


Nikhil M
To: Everyone
11:46 PM
can you plz explain why did we go with local cache instead of global cache?

- if cache is global: app server doesnt do alot by itself, so fuse!
- locking
- sharding: simply use app-server as indivdual shards!

Abhishek Pandey
To: Everyone
11:47 PM
what about distributed/central cache case ?

the consistent hashing is done such each shards handle like around 10m users in cases of FB?

yes recalculation done at 2:50 hrs - 1000 ap servers!

so there are two levels of consistent hashing one at app server cache level and another at db server level?

Since we shard secondary db by userid and it stores the summary of every conversation, how does systems like WhatsApp take care of this since it’ll have to write the same message for 256 users in the secondary db, on top of that these users can be in multiple shards

db never does hashing: whoever is talking to it does. app server or LB

- whatsapp has user limit. why? because groups aint common!

but look more!!!!

what protocol between client and the app server is used, websocket, grpc etc?

could you please summerise sharding part, in both cases (with secondary Db and without) we have write issues, which one we are using for FB messenger?

most messaging apps use websockets - a live persistent two way connections = http/https

- too many websockets is another problem to be handled!

- whatsapp can do more than 2mn web socket connections on a single server

for any convo with 1:1 mainly -  user id

if groups more common - convo-id
if convo-id: secondary db
- for user-convo mapping with shard = user-id
- this list can be sorted by recency or not 

- if sorted by recency: will have to update all the users' shards which are involved in the group chats, each time.

Cassandra has tunable consistency wrt to read and write like we can have strong consistency for write as compared to read, so isn't Cassandra not capable of handling such high i/o apps?

Since, we were sharding based on conversationId, there might be alot of shards that can get created.
Are there any cons associated with multiple shards ? (Ex - maintainability... or others?)


between cassandra and hbase - write throughput is more for hbase

cassandra's write throughput doesnt mean it gives the best consistency

its the R+W thing

-------------------------------------------------------no sharding is only a logical thing - no cons for maintiaing lots of shards

we are making size of cahce similar to db it is very costly becasue we are writing in ssd even it is more costly than db. can u claer that

cache in-memory: will store alot of data - but wont be stroing to the amount of 10yrs of data, right?

so it will still be ALOT smaller than db!
but compared to other caches - very huge

calculation: 140TB data in cache for 10 days
the point of caching is to make db visits rare!

99% of reads will be for last 10days only!

also cache wont be columnar thing right? it will prolly be key value or something.

DIY -> Now exactly what will you store ion cache!

could you tell how idempotence actually works? how will the server know if this is the same message that was sent before or another message with the same content? how will the server de-duplicate it? 

f(x) = f(f(x)) no matter how many times the fxn is applied

example: modulus, signum, abs

in terms of APIs:

GET: read request - can it ever be idempotent? until the data is guraranteed to be static - its NEVER idempotent

POST ( insert) / PUT (update) - 

- usually POST is USUALLY NOT idempotent.
- PUT is USUALLY idempotent - since it is modifying EXACT same data. 

- in our case - we make the post idempotent by setting a request id -> which keeps on retrying until request succeeds

- ideally

If message is in html format with image then how to handle storage. 

normally don't do that - so that injection problems may happen.

more importantly - its browsers problem. we can send simple text too

not clear on using seconday database whenvsharded key is Conversation id

Why GET method is not idempotent ? 

Also please explain POST & PUT are idempotent or not

How we store the messages on different servers and how will we retrieve that from that so quickly ?

how the secondary database is  solving the issue for particual userid?

is conversation id is nothing but groupid 

if message are huge how low latency is possible?

do we need custom consistent hashing algo at lb wrt local cache routing? does key range changes for shard in case of scaling for local cache?

regarding latency i am not claer can u help me in that case i know it is very basic u can take my ths question at last

if HTTP call are there then within 10 sec what response we will give them if fail and roll back then data lost sender need to send request again

this question can answered at the end, as its not directly related to the case study we discussed today. I am working on dividing monolith service into microservices, out of which if payment service is considered . with instant request and response between these services, how these services should talk to each other. Webhooks/rest api's or  messaging queue.


Payment, ecommerce, etc
- 15-20k active users -> scalability -> 1L MAU
- 10^6 requests per day
- 10 req/sec
- use fucking anything :) 

listen to this whole snip again :/

Bloom Filter - How to handle partial fail condition?

Is idempotence a varying feature for every app or is it same for all apps ? .i.e. here in this case of FB it is resending the message if not delivered. 

Does the WAL file store only data or only logs?
- data too! ofc! deltas

https://docs.google.com/document/d/1FE10Pu4nd6sz89RHq82rkcO2fdNoOZA0jQgO8pLgsOc/edit#