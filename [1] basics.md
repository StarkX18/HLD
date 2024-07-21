agenda:

1. what and why HLD?
2. case study: del.icio.us
3. scaling challenges
4. stateful vs stateless servers
5. consistent hashing
6. hld curriculum overview

Any reference books you can suggest?

- Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems
Book by Martin Kleppmann

- https://drive.google.com/drive/folders/1vNxeaGh376tit16Lo_mu2s9xIUP0vIEU?usp=sharing

https://github.com/jeffrey-xiao/papers/blob/master/textbooks/designing-data-intensive-applications.pdf

why?
- we generally work with softwares on a single machine - as devs
- maybe hundreds of request per second
- this scenario isnt the real case scenario!!!

- is billion users really a thing?
- FAANG, linkedin, whatsapp etc


for this scale: what becomes important is : architecting the code!
- HLD


as you go further in your career: you will move away from making code decisions to making design decisions!

- you will solve more and more complex engineers

Google SSE:
- a file stores a bunch of strings - search queries say
- you want to sort this file in ascending order - lexicographically.

--------------------------------------------------
constraint:
- 50pb of data = 50mn gb of data
--------------------------------------------------

no ram, no disk can store this!!!
you need multiple data centres!

suggestions????
solution: map reduce with external sorting

del.icio.us: 2003 - bookmarkign website
|
|
youtube: 2005
aws: 2006
chrome: 2008


whenever we discuss features - ONLY go for the mvp: minimum viable product or POC:

- user registration and signin/out: authentication and authorization - whats the difference? google!!
- save bookmark -> url, uid
- get all bookmarks - uid

fantastic websites and where to find them!

server : a device connected to net which primarily provides an internet service

---------- connected to internet------------
via ISP - internet service provider

- jio, tikona, bsnl, mtnl, airtel etc
- they provide an IP address
- IPv4 vs IPv6: DIY!

- the IP is dynamic in nature not static!
- why dynamic?? because subnets!
DIY
--------------------------
whenevr you want a SERVER - you REQUEST a STATIC ip addresss!!!!
---------------------------------------------------

but we NEVER type IP address!! we type strings!

The browser contacts the DNS server and asks for the IP of the website

DNSServer():
map[domain_name]:IP_address

ICANN: Non profit: stores the mapping of domain to ip
---------------------------------------------------

How does your browser KNOW what the ICANN IP is?

lets assume it is hardcoded. Its not but ok.
---------------------------------------------------

ICANN - provides a service that provides the IP for a given domain name.

domain name service, it is called.

anyways: these mappings are gonna be huge!

too many devices! too many websites! 

ICANN server crashes (say) -> entire internet goes down -> SPOF and overloaded


so better design: intermediary DNS servers!!!
--- the DNS servers sync their data from the source of truth vis-a-vix ICANN server

solbes SPOF and overloading

issues still:

1. ICANN server sstill goes down
2. sync times!
3. which dns and are all synced simultaneously?
4. which dns server?
5. so on - list down all you can think of!

server room -multiple servers with hot copy : a different server that maintains the exact same data as the original server!

its not exactly a master slave: any CUD will happen simultaneously on all hot copies. but reads will only happen from a single active copy. if it goesa down - some other hot copy becomes the active copy


so in case of hotcopy master slave will work

ISP->IP
DNS-> IP to server based on domain
Domain registrar: godaddy, namecheap, google domains


who benefits from DNS?
companies have vested interest in maintaining internet!
- google dns servers: 8.8.8.8, 8.8.8.4: static IP
- so they pay to get dns maintained
-------------------------------------------------
if dns server is far away: 100 ms of  latency, each side of roundtrip

- similarly jio is interested in maintaining  their own DNS servers too!
- all big companies, ISPs , millitary hosts DNS servers


router is connected to ISP and ISP tells it to use a particular DNS - its own prolly.

you can totally overwrite that! :P -> DO IT!

how does govt block websites? block it from DNS server itself!

Any reason behind you using google DNS server?

demos:

Pragy's google dns:
google.com -> IP adress: 172.217.67.4:143
- 143=ssl

terminal: 
- man traceroute.
- traceroute dbdb.io
- for every domain name: there are more than single IP addresses

0 - DNS call that happened is not shown!
1 - 192.168.1.1 -> router IP address
2 - ISP - a bunch of airtel servers
- whois or ipinfo 
3 - cloudflare server, in berlin! 
4. cloudflare server, in san francisco, california!

try for google

configuring dns: google domains-
very cheap!
- domains.google.com

deli.icio.us contd:
-----------------------------------------------
ram: 128 mb
disk 40gb
intel pentium duo core 3.2Ghz
-----------------------------------------------
things running:
db,
website server,
os, background tasks etc
------------------------------------------------
issues:
1. power issues: joshua sleeps, goes away, rain etc
2. overloaded? processor slow, too many tasks etc
3. bandwidth? network
------------------------------------------------
analysing disk space:

for user id:
1. 4bytes - pow(2,16) - 4Bn
2. 8bytes - pow(2,32)- 16BnBn
use (1)

for urls: lets say average 1000 chars: 1000 bytes
- 1kb

1mn bookmarks/day say: 1kb data per row = 1kb * 1Mn = 1GB data / day

40 days of data only!!!

how can joshua counter this bs?

- upgrade the computer: delays disaster not averts it: life lesson here!

vertical scaling - upgrading the servers until the world's end!

Buy more cheapest computers: google way.
- he is not limited by technology here - horizontal scaling

horizontal vs vertical scaling:
- vertical:easy, horizontal diff to manage 
   technically
- horizontal scaling is the reason for hld
- vertical scaling has a hard limit to how much you can scale! pricew, tech both exponential increase
- price grows sublinearly as economy of scale kicks in
- first thing to consider - vertical scaling
- horizontal scaling is the LAST!
because its difficult to manage!

scale up(v) vs scale out(h)

- aws: ec2 instance most expensive??!!
448 CPUs!
12TB of RAM!
100GB network!!!!

looks like a concept machine :)

vertical sclae is there any limit how much we can scale

now we have done horizontal scaling.

- what IP should our client connect to?

- which IP is registered for our dns? - mac address is different but IP is same because IP is tied to connection (mtnl landline example), not machine

- choose one laptop: let it distribute load!

- single point of failure? pick up another candidate from pool - make it easy swappable!

- overloaded? prolly no: very simple job - maybe round robin etc
- typical LB can handle upwards of 10mn requests per second!

- but for something like google this wont work!

- hence the multiple IP adddresses and DNS come into picture!

- multiple mappings in the DNS itself act as LB! - the client browser selects the IP - not the dns:

generally - geoDNS tells nearest location IP

LB needs to know which servers are online:

1. heartbeat:
- push based mechanism
- servers send a periodic signal to LB, "im alive"

2. healthcheck: 
- pull based mechanism
- the LB sends requests to each server periodically to check if its alive.

suppose we have multiple load balance addresses
how to identify which load balancer is going to server client request if we have multiple load balancer in place, are we using any kind of algo?

if LB gies down - another machine tkes its place from the server pool!

so, HLD also handles disaster recorvery? yes

LB exercise 2 mechanisms:
Heartbeat and Healthcheck mechanisms:

heartbeat -> push mechanism likes heart -> sends signal -> if stops coming for some time -> assume dead

healthcheck -> pull mechanism like health checkup -> doctor checks if I am alive...


Healthcheck configs:
ping protocol - http, ssl etc,
ping port - which port to ping,
ping path - route to ping,
response timeout: how much time till healthcheck fail,
interval: between healthchecks,
unhealthy threshold: how many healthchecks before declaring down,
healthy threshold: same as unhealthy

there are multiple approaches how to load balancers will distribute the load laters..

Would the path to server will be variable here?yes

I heard there is client/server side loadbalancing. Will we be discussing them?

left some portions - 

hld curriculum:

1. hld 1o1 + consistent hashing
2. caching - seperate BL from data
3.  caching contd
4. CAP theorem 
5. master slave
6. sql vs nosql
7. nosql
8. case studies - type ahead, 
9. multi master, 
10. messaging apps
11. notifications 
12. zookeepers and messaging queues and rabbitmq - theoritically
13. elasticsearch
14. uber/zomato: nearest neighbour queries: quad trees
15. file storage systems(s3)
16. uber (case study)
17. spl class: popular interview questions: GC, Rate limiter etc
18. hotstar -> Mudit
19. Microservices
20. Case study: irctc and cuncurrency
-------------------------------------------------

cheat sheets: db

ChatGPT!!!

- GPT 3.5 + RL pipeline
- LLM transformer: GPT3
- RL pipeline
- 175 bn params = 700gb space of RAM, NOT disk soace!!!

LB - research paper!

https://docs.google.com/document/d/1DxQzLpu1XPe_mRWsewNWtKL6E4uwKHQhBp7GX6Sg7qI/edit

7351769231
pragy@scaler.com :)

https://docs.google.com/document/d/1DxQzLpu1XPe_mRWsewNWtKL6E4uwKHQhBp7GX6Sg7qI/edit

https://drive.google.com/drive/folders/1vNxeaGh376tit16Lo_mu2s9xIUP0vIEU?usp=sharing