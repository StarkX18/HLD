Starting caching:
---------------------------------------------
appserver layer:
- currently have data and biz logic on same server
- tightly coupled

- whats the probelm with tightly xouplwd?
- deploying the code - server down
- ci/cd happens 1k times everyday

- webserver wants :
- powerful cpu,
- high ram
- fast internet

- db wants:
- large disk space
- fast i/o

insanely costly server!!!
- plus STILL the two might interfere with each other
- we need dedicsated db and servers

split the db and app servers:
- multiple app-servers
- multiple db layers

---------------------------------------------------
- complexity: deal with...
- latency: optimise for...
- LB doesnt care which appserver to send the request to
- app-server in this case are stateless
- are problem solved by consistent hashing is STRICTLY for stateful servers!

- exception case of stateful servers? 
- authentication/authorization and session mgmt servers are stateful
- maintaining local cache is rqd
--------------------------------------------------

- but again with stateless server, data is still sharded and app-server needs to figure out which db to use! -> uses consistent hashing!

- LB is simply round robin
---------------------------------------------------
hbase/sql library: implements consisitent hashing which is why you dont need to implement!
- sends request to zookeeper
zookeeper: cluster manager - 

heartbeat or health check
---------------------------------------------------

so now the only job of LB is round robin? which is (++ctr)%(num_servers)? isnt that too less of a job for a whole server??

cooking example: supermarket vs fridge?!

- slow vs fast access
- only frequently used items in fridge but everything in supermarket
- supermarket alwasy fresh - fridge might have stale/rot

- with your body - which also caches nutrients in fat cells and muscles!
- when this cache invalidated -> fridge cache

- multiple levels of cache
in computer architecture too
---------------------------------------------------

caches in real life:

1. dns cache
2. static media: css, html, js, images
3. cookies/ local storage/ indexedDB
    - username, theme, config, profile pic, lang etc
4. CDN: another level of cache - for far off data

cloudflare, akamai, amazon cloudfront, fastly -> 375000 servers across the globe - cloudflare

----- CDNs reduce latency -------
- like in image resizer: LRU cache

- db content changes more frequesntly
- db queries + read/write ops very complex
----------------------------------------------

how does browser know which CDN to go to?

1. geoDNS: also gives location: but also not widely adopted
2. anycast: widely used: randomly hot a server -> CDN servers rerout/redirect (301 response) you to your nearest CDN because ofc they know each other's locatioin -> the other server -> data!
-- auth in cdns? auth urls - no auth otherwise
--------------------------------------------------------

but doesnt the brtowser cache the first request for its dns-ip mapping?
---------------------------------------------------

why is this re-routing even faster than our randomly hit CDN server getting our request?

also, why does thew server not directly send the request to other server, which can respond to us?

how do CDNs get data?
- <img src="<cdn-url>">
- scaler CDN is still authewnticated. how?

now we have LB -> app-server -> db-layer

app-servers have their own local cache - is it stateful? yes and no.
- yes: if random requests start coming in: cache wont be useful -  stateless kind
- no: if request carefully routed: cache utilised!

- local cache is on app-server,
- but we can even introduce  WHOLE CACHING LAYER ST: 
LB->app-server-layer->cache-layer->db-layer


use cases:
1. how ius local cache helpful? news feed iin fb

challenges with caching-

1. its not a source of truth. if stale data is ok -> this is ok, but still periodically invalidate the data.
2. limited in size -> data eviction

Hashmap+heap or hashmap + ll

CDNs: stroe some media files/heavy data like html, css, profile pictures, logos, videos etc

- vercel/heroku/netify? - 
- aws/cloudflare CDNs -> store a url like 
- cdn.cloudflare.com/1/abe46...some hash

- cdn doesnt do authentication or authorization
- however its url may be protected behing your own auth url
- auth will again happen with your account

- IIT Bombay: internal CDN -> instant streaming!!!
- orgs do maintain their own CDNs


q1) regarding anycast, what happens for second request onwards? does request still go to random cdn? or it will go to nearest cdn?

q2) how does cdn provider charges? does it depend upon resource size? or request count?
e.g. netflix is hosting videos over cdn, how it is generally charged to netflix in case netflix is using third party cdn servers?

CDN charges:
network traffic + data transfer across tje cdn boundary

akamai - 12 percent of internet!!


in this way any app which has login is stateful right?

session tokens etc in django etc not good design
auth and auth: dedicated auto serrvers: like gotrue

-> Personally have been using JWTs for stateless auth.
-> malkes server stateless: con cant revoke jwt..see!!! again