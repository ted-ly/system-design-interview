## Table of Content

* [What is system design?](#what-is-system-design)
* [Basic things you should know](#basic-things-you-should-know)
* [Step by step how to approach a system design interview question](#step-by-step-how-to-approach-a-system-design-interview-question)
* [Appendix](#appendix)
* [Company engineering blogs](#company-engineering-blogs)
* [Credits and resources](#credits-and-resources)

## What is system design?

The process of defining the architecture, modules, interfaces, and data for a system to satisfy specified requirements. [System design](https://en.wikipedia.org/wiki/Systems_design) could be seen as the application of systems theory to product environment.


## Basic things you should know before going for a interview

* [Availability patterns](#availability-patterns)
* [Domain name system](#domain-name-system)
* [Content delivery network](#content-delivery-network)
* [CAP theorem](#cap-theorem)
* [Load balancer](#load-balancer)
* [Reverse proxy](#reverse-proxy)
* [Caching](#caching)
* [Sharding](#sharding)
* [Queues](#queues)
* [SQL or noSQL](#sql-or-nosql)
* [Long-Polling vs websockets](#long-polling-vs-websockets)

## Step by step how to approach a system design interview question

### Step 1: Constraints and use cases

Gather as much information as possible before starting your design

* Clarify the system's constraints
* Identify what use cases the system needs to satisfy
* Questioning your interviewer and agreeing on the scope of the system
* Never assume things that were not explicitly stated
* How many users does your system serve?
* What are the I/O(input/output) of the system?
* How many requests per second do we expect?
* How does your system handle if more users use your service?
* What is the expected read to write ratio?
* How large is the data? How can we handle them?

### Step 2: Draw a high level design

Outlining all the important components that your architecture will need

* Draw a simple diagram of your ideas
* Sketch main components and the connections between them
* Justify your high-level design diagram
* Use well-known developed techniques

### Step 3: Design core components

Dive into details for each core component.  For example, if you were asked to design a url shortening service, discuss:

* Generating and storing a hash of the full url
    * MD5 and Base62
    * Hash collisions
    * SQL or NoSQL
    * Database schema
* Translating a hashed url to the full url
    * Database lookup
* API and object-oriented design

### Step 4: Scale the design

Identify and address bottlenecks, given the constraints.  For example, do you need the following to address scalability issues?

* Load balancer
* Horizontal scaling
* Caching
* Database sharding

Discuss potential solutions and trade-offs.  Everything is a trade-off.

### Availability patterns

There are two main patterns to support high availability: **fail-over** and **replication**.

### Fail-over

#### Active-passive

With active-passive fail-over, heartbeats are sent between the active and the passive server on standby.  If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.

The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby.  Only the active server handles traffic.

Active-passive failover can also be referred to as master-slave failover.

#### Active-active

In active-active, both servers are managing traffic, spreading the load between them.

If the servers are public-facing, the DNS would need to know about the public IPs of both servers.  If the servers are internal-facing, application logic would need to know about both servers.

Active-active failover can also be referred to as master-master failover.

#### Disadvantage(s): failover

* Fail-over adds more hardware and additional complexity.
* There is a potential for loss of data if the active system fails before any newly written data can be replicated to the passive.

#### Replication

Master-slave and master-master

* Master-slave replication
  * Master serves reads and writes, replicating writes to one or more slaves.
  * Slaves serves only reads, can replicate to additional slaves in a tree-like fashion.
  * If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned.
  * **Disadvanges(s):**
    * Additional logic is needed to promote a slave to a master.

* Master-master replication
  * Both masters serve reads and writes and coordinate with each other on writes.
  * If either master goes down, the system can continue to operate with both reads and writes.
  * **Disadvantage(s):**
    * You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
    * Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
    * Conflict resolution comes more into play as more write nodes are added and as latency increases.

* **Replication disadvantage(s):**
  * There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
  * Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
  * The more read slaves, the more you have to replicate, which leads to greater replication lag.
  * On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
  * Replication adds more hardware and additional complexity.


### Domain name system

<p align="center">
  <img src="http://i.imgur.com/IOyLj4i.jpg">
  <br/>
  <i><a href=http://www.slideshare.net/srikrupa5/dns-security-presentation-issa>Source: DNS security presentation</a></i>
</p>

A Domain Name System (DNS) translates a domain name such as www.example.com to an IP address.

DNS is hierarchical, with a few authoritative servers at the top level.  Your router or ISP provides information about which DNS server(s) to contact when doing a lookup.  Lower level DNS servers cache mappings, which could become stale due to DNS propagation delays.  DNS results can also be cached by your browser or OS for a certain period of time, determined by the [time to live (TTL)](https://en.wikipedia.org/wiki/Time_to_live).

* **NS record (name server)** - Specifies the DNS servers for your domain/subdomain.
* **MX record (mail exchange)** - Specifies the mail servers for accepting messages.
* **A record (address)** - Points a name to an IP address.
* **CNAME (canonical)** - Points a name to another name or `CNAME` (example.com to www.example.com) or to an `A` record.

Services such as [CloudFlare](https://www.cloudflare.com/dns/) and [Route 53](https://aws.amazon.com/route53/) provide managed DNS services.  Some DNS services can route traffic through various methods:

* [Weighted round robin](http://g33kinfo.com/info/archives/2657)
    * Prevent traffic from going to servers under maintenance
    * Balance between varying cluster sizes
    * A/B testing
* Latency-based
* Geolocation-based

### Disadvantage(s): DNS

* Accessing a DNS server introduces a slight delay, although mitigated by caching described above.
* DNS server management could be complex, although they are generally managed by [governments, ISPs, and large companies](http://superuser.com/questions/472695/who-controls-the-dns-servers/472729).
* DNS services have recently come under [DDoS attack](http://dyn.com/blog/dyn-analysis-summary-of-friday-october-21-attack/), preventing users from accessing websites such as Twitter without knowing Twitter's IP address(es).

### Source(s) and further reading

* [DNS architecture](https://technet.microsoft.com/en-us/library/dd197427(v=ws.10).aspx)
* [Wikipedia](https://en.wikipedia.org/wiki/Domain_Name_System)
* [DNS articles](https://support.dnsimple.com/categories/dns/)

### Content delivery network

<p align="center">
  <img src="http://i.imgur.com/h9TAuGI.jpg">
  <br/>
  <i><a href=https://www.creative-artworks.eu/why-use-a-content-delivery-network-cdn/>Source: Why use a CDN</a></i>
</p>

A content delivery network (CDN) is a globally distributed network of proxy servers, serving content from locations closer to the user.  Generally, static files such as HTML/CSS/JS, photos, and videos are served from CDN, although some CDNs such as Amazon's CloudFront support dynamic content.  The site's DNS resolution will tell clients which server to contact.

Serving content from CDNs can significantly improve performance in two ways:

* Users receive content at data centers close to them
* Your servers do not have to serve requests that the CDN fulfills

### Push CDNs

Push CDNs receive new content whenever changes occur on your server.  You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN.  You can configure when content expires and when it is updated.  Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.

Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs.  Content is placed on the CDNs once, instead of being re-pulled at regular intervals.

### Pull CDNs

Pull CDNs grab new content from your server when the first user requests the content.  You leave the content on your server and rewrite URLs to point to the CDN.  This results in a slower request until the content is cached on the CDN.

A [time-to-live (TTL)](https://en.wikipedia.org/wiki/Time_to_live) determines how long content is cached.  Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.

Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.

### Disadvantage(s): CDN

* CDN costs could be significant depending on traffic, although this should be weighed with additional costs you would incur not using a CDN.
* Content might be stale if it is updated before the TTL expires it.
* CDNs require changing URLs for static content to point to the CDN.

### Source(s) and further reading

* [Globally distributed content delivery](http://repository.cmu.edu/cgi/viewcontent.cgi?article=2112&context=compsci)
* [The differences between push and pull CDNs](http://www.travelblogadvice.com/technical/the-differences-between-push-and-pull-cdns/)
* [Wikipedia](https://en.wikipedia.org/wiki/Content_delivery_network)

### CAP theorem

<p align="center">
  <img src="http://i.imgur.com/bgLMI2u.png">
  <br/>
  <i><a href=http://robertgreiner.com/2014/08/cap-theorem-revisited>Source: CAP theorem revisited</a></i>
</p>

In a distributed computer system, you can only support two of the following guarantees:

* **Consistency** - Every read receives the most recent write or an error
* **Availability** - Every request receives a response, without guarantee that it contains the most recent version of the information
* **Partition Tolerance** - The system continues to operate despite arbitrary partitioning due to network failures

*Networks aren't reliable, so you'll need to support partition tolerance.  You'll need to make a software tradeoff between consistency and availability.*

#### CP - consistency and partition tolerance

Waiting for a response from the partitioned node might result in a timeout error.  CP is a good choice if your business needs require atomic reads and writes.

#### AP - availability and partition tolerance

Responses return the most recent version of the data available on the a node, which might not be the latest.  Writes might take some time to propagate when the partition is resolved.

AP is a good choice if the business needs allow for [eventual consistency](#eventual-consistency) or when the system needs to continue working despite external errors.

### Source(s) and further reading

* [CAP theorem revisited](http://robertgreiner.com/2014/08/cap-theorem-revisited/)
* [A plain english introduction to CAP theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem/)
* [CAP FAQ](https://github.com/henryr/cap-faq)

### Load balancer

<p align="center">
  <img src="http://i.imgur.com/h81n9iK.png">
  <br/>
  <i><a href=http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html>Source: Scalable system design patterns</a></i>
</p>

Load balancers distribute incoming client requests to computing resources such as application servers and databases.  In each case, the load balancer returns the response from the computing resource to the appropriate client.  Load balancers are effective at:

* Preventing requests from going to unhealthy servers
* Preventing overloading resources
* Helping eliminate single points of failure

Load balancers can be implemented with hardware (expensive) or with software such as HAProxy.

Additional benefits include:

* **SSL termination** - Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
    * Removes the need to install [X.509 certificates](https://en.wikipedia.org/wiki/X.509) on each server
* **Session persistence** - Issue cookies and route a specific client's requests to same instance if the web apps do not keep track of sessions

To protect against failures, it's common to set up multiple load balancers, either in [active-passive](#active-passive) or [active-active](#active-active) mode.

Load balancers can route traffic based on various metrics, including:

* Random
* Least loaded
* Session/cookies
* [Round robin or weighted round robin](http://g33kinfo.com/info/archives/2657)
* [Layer 4](#layer-4-load-balancing)
* [Layer 7](#layer-7-load-balancing)

### Layer 4 load balancing

* Layer 4 load balancing uses information defined at the networking transport layer for deciding how to distribute client requests across a group of servers.
* Layer 4 bases load-balancing decision on the source and destination IP addresses and ports recorded in the packing header, without considering the contents of the packet.
* When it makes load balancing decision, it also performs Network Address Translation (NAT) on the request packet, changing the recorded destination IP address from its own to that of the content server it has chosen on the internal network.
* Similarly, before forwarding server responses to clients, the load balancer changes the source address recorded in the packet header form the server's IP address to its own.
* Layer 4 load balancers make their routing decisions based on address information extracted from the first few packets in the Transmission Control Protocol (TCP) stream, and do not inspect the packet content.
* A layer 4 load balancer is often a dedicated hardware device supplied by a vendor and runs proprietary load-balancing software, and NAT operations might be performed by specialized chips rather than in software.
* Layer 4 load balancing was a popular architectural approach to traffic handling when computer hardware was not as powerful as it is today, and the interaction between clients and application servers was much less complex.

### Layer 7 load balancing

* Layer 7 load balancing operates at high-level application layer, which deals with actual content of each message.
* HTTP is predominant Layer 7 protocol for website traffic on the internet.
* A Layer 7 load balancer terminates the network traffic and reads the message within, it can make load-balancing decision based on the content of the message (the URL or cookie, for example).  It then makes a new TCP connection to the selected upstream server (or reuses the existing one, by means of [HTTP keepalives](https://www.nginx.com/blog/http-keepalives-and-web-performance/)) and writes the request to the server.
* Layer 7 load balancing is more CPU-intensive than packet-based Layer 4 load balancing, but rarely causes degraded performance on a modern server.
* Layer 7 load balancing enables the load balancer to make smarter load-balancing decisions, and to apply optimizations and changes to the content (such as compression and encryption). It uses buffering to offload slow connections from the upstream servers, which improves performance.
* A device that performs Layer 7 load balancing is often referred to as a [reverse-proxy](#reverse-proxy) server.

### Horizontal scaling

Load balancers can also help with horizontal scaling, improving performance and availability.  Scaling out using commodity machines is more cost efficient and results in higher availability than scaling up a single server on more expensive hardware, called **Vertical Scaling**.  It is also easier to hire for talent working on commodity hardware than it is for specialized enterprise systems.

#### Disadvantage(s): horizontal scaling

* Scaling horizontally introduces complexity and involves cloning servers
    * Servers should be stateless: they should not contain any user-related data like sessions or profile pictures
    * Sessions can be stored in a centralized data store such as a [database](#database) (SQL, NoSQL) or a persistent [cache](#cache) (Redis, Memcached)
* Downstream servers such as caches and databases need to handle more simultaneous connections as upstream servers scale out

### Disadvantage(s): load balancer

* The load balancer can become a performance bottleneck if it does not have enough resources or if it is not configured properly.
* Introducing a load balancer to help eliminate single points of failure results in increased complexity.
* A single load balancer is a single point of failure, configuring multiple load balancers further increases complexity.

### Source(s) and further reading

* [NGINX architecture](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
* [HAProxy architecture guide](http://www.haproxy.org/download/1.2/doc/architecture.txt)
* [Scalability](http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones)
* [Wikipedia](https://en.wikipedia.org/wiki/Load_balancing_(computing))
* [Layer 4 load balancing](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)
* [Layer 7 load balancing](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)
* [ELB listener config](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-listener-config.html)

### Reverse proxy

* A reverse proxy server is a type of proxy server that typically sits behind the firewall in a private network and directs client requests to the appropriate backend server.
* A reverse proxy provides an additional level of abstraction and control to ensure the smooth flow of network traffic between clients and servers.
* Benefits:
  * **Increased security** - No information about backend servers is visible outside internal network, so malicious clients cannot access them directly to exploit any vulnerabilities. Many reverse proxy servers include features that help protect backend servers from [distributed denial-of-service](https://www.nginx.com/resources/glossary/distributed-denial-of-service/) (DDoS) attacks, for example by rejecting traffic from particular client IP addresses(blacklisting), or limiting the number of connections accepted from each client.
  * **Increased scalability and flexibility** - Because clients see only the reverse proxy's IP address, you are free to change the configuration of your backend infrastructure. This is particularly useful in a load-balanced environment, where you can scale the number of severs up and down to match fluctuations in traffic volume.
  * **Web acceleration** - reducing the time it takes to generate a response and return it to the client.  Techniques for web acceleration include the following:
    * **Compression** - Compressing server responses before returning them to the client (for instance with gzip) reduces the amount of bandwidth they require, which speeds their transit over the network.
    * **SSL termination** - Encrypting the traffic between clients and servers protects it as it crosses a public network like the internet. But decryption and encryption can be computationally expensive.  By decrypting incoming requests and encrypting server responses, the reverse proxy frees up resources on backend servers which they can then devote to their main purpose, serving content.
    * **Caching** - Before returning the backend server's response to the client, the reverse proxy stores a copy of it locally. When the client (or any client) makes the same request, the reverse proxy can provid the response itself from the cache instead of forwarding the request to the backend server. This both decreases response time to the client and reduces the load on the backend server.
    * **Static content** - Serve static content directly
      * HTML/CSS/JS
      * Photos
      * Videos
      * Etc

### Load balancer vs reverse proxy

* Deploying a load balancer is useful when you have multiple servers.  Often, load balancers  route traffic to a set of servers serving the same function.
* Reverse proxies can be useful even with just one web server or application server, opening up the benefits described in the previous section.
* Solutions such as NGINX and HAProxy can support both layer 7 reverse proxying and load balancing.

### Disadvantage(s): reverse proxy

* Introducing a reverse proxy results in increased complexity.
* A single reverse proxy is a single point of failure, configuring multiple reverse proxies (ie a [failover](https://en.wikipedia.org/wiki/Failover)) further increases complexity.

### Source(s) and further reading

* [Reverse proxy vs load balancer](https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/)
* [NGINX architecture](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
* [HAProxy architecture guide](http://www.haproxy.org/download/1.2/doc/architecture.txt)
* [Wikipedia](https://en.wikipedia.org/wiki/Reverse_proxy)

### Caching

<p align="center">
  <img src="http://i.imgur.com/Q6z24La.png">
  <br/>
  <i><a href=http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html>Source: Scalable system design patterns</a></i>
</p>

Caching improves page load times and can reduce the load on your servers and databases.  In this model, the dispatcher will first lookup if the request has been made before and try to find the previous result to return, in order to save the actual execution.

Databases often benefit from a uniform distribution of reads and writes across its partitions.  Popular items can skew the distribution, causing bottlenecks.  Putting a cache in front of a database can help absorb uneven loads and spikes in traffic.

### Client caching

Caches can be located on the client side (OS or browser), [server side](#reverse-proxy), or in a distinct cache layer.

### CDN caching

[CDNs](#content-delivery-network) are considered a type of cache.

### Web server caching

[Reverse proxies](#reverse-proxy-web-server) and caches such as [Varnish](https://www.varnish-cache.org/) can serve static and dynamic content directly.  Web servers can also cache requests, returning responses without having to contact application servers.

### Database caching

Your database usually includes some level of caching in a default configuration, optimized for a generic use case.  Tweaking these settings for specific usage patterns can further boost performance.

### Application caching

In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage.  Since the data is held in RAM, it is much faster than typical databases where data is stored on disk.  RAM is more limited than disk, so [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms) algorithms such as [least recently used (LRU)](https://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used) can help invalidate 'cold' entries and keep 'hot' data in RAM.

Redis has the following additional features:

* Persistence option
* Built-in data structures such as sorted sets and lists

There are multiple levels you can cache that fall into two general categories: **database queries** and **objects**:

* Row level
* Query-level
* Fully-formed serializable objects
* Fully-rendered HTML

Generally, you should try to avoid file-based caching, as it makes cloning and auto-scaling more difficult.

### Caching at the database query level

Whenever you query the database, hash the query as a key and store the result to the cache.  This approach suffers from expiration issues:

* Hard to delete a cached result with complex queries
* If one piece of data changes such as a table cell, you need to delete all cached queries that might include the changed cell

### Caching at the object level

See your data as an object, similar to what you do with your application code.  Have your application assemble the dataset from the database into a class instance or a data structure(s):

* Remove the object from cache if its underlying data has changed
* Allows for asynchronous processing: workers assemble objects by consuming the latest cached object

Suggestions of what to cache:

* User sessions
* Fully rendered web pages
* Activity streams
* User graph data

### When to update the cache

Since you can only store a limited amount of data in cache, you'll need to determine which cache update strategy works best for your use case.

#### Cache-aside

<p align="center">
  <img src="http://i.imgur.com/ONjORqk.png">
  <br/>
  <i><a href=http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast>Source: From cache to in-memory data grid</a></i>
</p>

The application is responsible for reading and writing from storage.  The cache does not interact with storage directly.  The application does the following:

* Look for entry in cache, resulting in a cache miss
* Load entry from the database
* Add entry to cache
* Return entry

```
def get_user(self, user_id):
    user = cache.get("user.{0}", user_id)
    if user is None:
        user = db.query("SELECT * FROM users WHERE user_id = {0}", user_id)
        if user is not None:
            key = "user.{0}".format(user_id)
            cache.set(key, json.dumps(user))
    return user
```

[Memcached](https://memcached.org/) is generally used in this manner.

Subsequent reads of data added to cache are fast.  Cache-aside is also referred to as lazy loading.  Only requested data is cached, which avoids filling up the cache with data that isn't requested.

##### Disadvantage(s): cache-aside

* Each cache miss results in three trips, which can cause a noticeable delay.
* Data can become stale if it is updated in the database.  This issue is mitigated by setting a time-to-live (TTL) which forces an update of the cache entry, or by using write-through.
* When a node fails, it is replaced by a new, empty node, increasing latency.

#### Write-through

<p align="center">
  <img src="http://i.imgur.com/0vBc0hN.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

The application uses the cache as the main data store, reading and writing data to it, while the cache is responsible for reading and writing to the database:

* Application adds/updates entry in cache
* Cache synchronously writes entry to data store
* Return

Application code:

```
set_user(12345, {"foo":"bar"})
```

Cache code:

```
def set_user(user_id, values):
    user = db.query("UPDATE Users WHERE id = {0}", user_id, values)
    cache.set(user_id, user)
```

Write-through is a slow overall operation due to the write operation, but subsequent reads of just written data are fast.  Users are generally more tolerant of latency when updating data than reading data.  Data in the cache is not stale.

##### Disadvantage(s): write through

* When a new node is created due to failure or scaling, the new node will not cache entries until the entry is updated in the database.  Cache-aside in conjunction with write through can mitigate this issue.
* Most data written might never read, which can be minimized with a TTL.

#### Write-behind (write-back)

<p align="center">
  <img src="http://i.imgur.com/rgSrvjG.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

In write-behind, the application does the following:

* Add/update entry in cache
* Asynchronously write entry to the data store, improving write performance

##### Disadvantage(s): write-behind

* There could be data loss if the cache goes down prior to its contents hitting the data store.
* It is more complex to implement write-behind than it is to implement cache-aside or write-through.

#### Refresh-ahead

<p align="center">
  <img src="http://i.imgur.com/kxtjqgE.png">
  <br/>
  <i><a href=http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast>Source: From cache to in-memory data grid</a></i>
</p>

You can configure the cache to automatically refresh any recently accessed cache entry prior to its expiration.

Refresh-ahead can result in reduced latency vs read-through if the cache can accurately predict which items are likely to be needed in the future.

##### Disadvantage(s): refresh-ahead

* Not accurately predicting which items are likely to be needed in the future can result in reduced performance than without refresh-ahead.

### Disadvantage(s): cache

* Need to maintain consistency between caches and the source of truth such as the database through [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms).
* Need to make application changes such as adding Redis or memcached.
* Cache invalidation is a difficult problem, there is additional complexity associated with when to update the cache.

### Source(s) and further reading

* [From cache to in-memory data grid](http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast)
* [Scalable system design patterns](http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html)
* [Introduction to architecting systems for scale](http://lethain.com/introduction-to-architecting-systems-for-scale/)
* [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
* [Scalability](http://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache)
* [AWS ElastiCache strategies](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Strategies.html)
* [Wikipedia](https://en.wikipedia.org/wiki/Cache_(computing))

### Sharding

* Sharding distributes data across different databases such that each database can only manage a subset of a data.
* Sharding results in less read and write traffic, less replication, and more cache hits.
* Index size is also reduced, which generally improves performances with faster queries.
* If one shard goes down, the other shards are still operational, add some form of replication to avoid data loss.
* Common ways to shard a table of users is either through the user's last name initial or the user's geographical location.

### Disadvantage(s):

* You'll need to update your application logic to work with shards, which could result in complex SQL queries.
* Data distribution can become lopsided in a shard. For example, a set of power users on a shard could result in increased load to that shard compared to others.
  * Rebalancing adds additional complexity. A sharding function based on consistent hashing can reduce the amount of transfered data.
* Joining data from multiple shards is more complex.
* Sharding adds more hardware and additional complexity.

#### Source(s) and further reading:

* [The coming of the shard](http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html)
* [Shard database architecture](https://en.wikipedia.org/wiki/Shard_(database_architecture))
* [Consistent hashing](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html)

### Queues

#### Message queues

* Message queues receive, hold, and deliver messages.
* How to use message queue in your application:
  * An application publishes a job to the queue, then notifies the user of job status
  * A worker picks up the job from the queue, processes it, then signals the job is complete
* User is not blocked and the job is processed in the background. During this time, the client might optionally do a small amount of processing to make it seem like the task has completed.
* **Redis** is useful as a simple message broker but messages can be lost.
* **RabbitMQ** is popular but requires you to adapt to the AMQP protocol and manage your own nodes.
* **Amazon SQS** is hosted but can have high latency and has the possibility of messages being delivered twice.

#### Task queues

* Task queues receive tasks and their related data, runs them, then delivers their results.
* Can support scheduling and can be used to run computationally-intense jobs in the background.
* **Celery** has support for scheduling and primarily has python support.

#### Back pressure

* Queues can grow significantly over time, the size can become larger than memory, resulting in cache misses, disk reads, and even slower performance.
* Back pressure can help by limiting the queue size, maintain a high throughput rate and good response times for jobs already in the queue.
* If the queue fills up, clients will get a server busy or HTTP 503 status code to try again later. Clients can retry the request at a later time.

### Disadvantage(s):

* Use cases such as inexpensive calculations and realtime workflows might be better suited for synchronous operations, as introducing queues can add delays and complexity.

#### Source(s) and further reading:

* [It's all a numbers game](https://www.youtube.com/watch?v=1KRYH75wgy4)
* [Applying back pressure when overloaded](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html)
* [Little's law](https://en.wikipedia.org/wiki/Little%27s_law)
* [What is the difference between a message queue and a task queue?](https://www.quora.com/What-is-the-difference-between-a-message-queue-and-a-task-queue-Why-would-a-task-queue-require-a-message-broker-like-RabbitMQ-Redis-Celery-or-IronMQ-to-function)

### SQL or noSQL

Reasons for SQL:

* Structured data
* Strict schema
* Relational data
* Need for complex joins
* Transactions
* Clear patterns for scaling
* More established: developers, community, code, tools, etc
* Lookups by index are very fast

Reasons for NoSQL:

* Semi-structured data
* Dynamic or flexible schema
* Non-relational data
* No need for complex joins
* Store many TB (or PB) of data
* Very data intensive workload
* Very high throughput for IOPS

Sample data well-suited for NoSQL:

* Rapid ingest of clickstream and log data
* Leaderboard or scoring data
* Temporary data, such as a shopping cart
* Frequently accessed ('hot') tables
* Metadata/lookup tables

#### Source(s) and further reading:

* [Scaling up to your first 10 million users](https://www.youtube.com/watch?v=vg5onp8TU6Q)
* [SQL vs NoSQL differences](https://www.sitepoint.com/sql-vs-nosql-differences/)

### Long-Polling vs websockets

#### Long-Polling

<p align="center">
  <img src="https://i.stack.imgur.com/xuzzd.png">
</p>

A variation of the traditional polling technique and allows emulation of an information push from a server to a client. With long polling, the client requests information from the server in a similar way to a normal poll.

* If the server does not have any information available for the client, instead of sending an empty response, the server holds the request and waits for some information to be available.
* Once the information becomes available (or after a suitable timeout), a complete response is sent to the client. The client will normally then immediately re-request information from the server, so that the server will almost always have an available waiting request that it can use to deliver data in response to an event.
* In a web/AJAX context, long polling is also known as Comet programming.

#### Websockets

<p align="center">
  <img src="https://i.stack.imgur.com/ibBGL.png">
</p>

WebSockets provide a persistent connection between a client and server that both parties can use to start sending data at any time.

* The client establishes a WebSocket connection through a process known as the WebSocket handshake. This process starts with the client sending a regular HTTP request to the server.
* An Upgrade header is included in this request that informs the server that the client wishes to establish a WebSocket connection.

#### Source(s) and further reading:

* [What are Long-Polling, Websockets, Server-Sent Events (SSE) and Comet?](https://stackoverflow.com/questions/11077857/what-are-long-polling-websockets-server-sent-events-sse-and-comet)
* [WebSockets vs Server-Sent Events vs Long-polling](http://dsheiko.com/weblog/websockets-vs-sse-vs-long-polling/)
* [Differences between websockets and long polling for turn based game server](https://stackoverflow.com/questions/31715179/differences-between-websockets-and-long-polling-for-turn-based-game-server)
* [An Introduction to WebSockets](http://blog.teamtreehouse.com/an-introduction-to-websockets)
* [Push technology](https://en.wikipedia.org/wiki/Push_technology#Long_polling)

### Back-of-the-envelope calculations

You might be asked to do some estimates by hand.  Refer to the [Appendix](#appendix) for the following resources:

* [Use back of the envelope calculations](http://highscalability.com/blog/2011/1/26/google-pro-tip-use-back-of-the-envelope-calculations-to-choo.html)
* [Powers of two table](#powers-of-two-table)
* [Latency numbers every programmer should know](#latency-numbers-every-programmer-should-know)

### Source(s) and further reading

Check out the following links to get a better idea of what to expect:

* [How to ace a systems design interview](https://www.palantir.com/2011/10/how-to-rock-a-systems-design-interview/)
* [The system design interview](http://www.hiredintech.com/system-design)
* [Intro to Architecture and Systems Design Interviews](https://www.youtube.com/watch?v=ZgdS0EUmn70)

## Appendix

You'll sometimes be asked to do 'back-of-the-envelope' estimates.  For example, you might need to determine how long it will take to generate 100 image thumbnails from disk or how much memory a data structure will take.  The **Powers of two table** and **Latency numbers every programmer should know** are handy references.

### Powers of two table

```
Power           Exact Value         Approx Value        Bytes
---------------------------------------------------------------
7                             128
8                             256
10                           1024   1 thousand           1 KB
16                         65,536                       64 KB
20                      1,048,576   1 million            1 MB
30                  1,073,741,824   1 billion            1 GB
32                  4,294,967,296                        4 GB
40              1,099,511,627,776   1 trillion           1 TB
```

#### Source(s) and further reading

* [Powers of two](https://en.wikipedia.org/wiki/Power_of_two)

### Latency numbers every programmer should know

```
Latency Comparison Numbers
--------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                          100   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy            10,000   ns       10 us
Send 1 KB bytes over 1 Gbps network     10,000   ns       10 us
Read 4 KB randomly from SSD*           150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from 1 Gbps  10,000,000   ns   10,000 us   10 ms  40x memory, 10X SSD
Read 1 MB sequentially from disk    30,000,000   ns   30,000 us   30 ms 120x memory, 30X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
```

Handy metrics based on numbers above:

* Read sequentially from disk at 30 MB/s
* Read sequentially from 1 Gbps Ethernet at 100 MB/s
* Read sequentially from SSD at 1 GB/s
* Read sequentially from main memory at 4 GB/s
* 6-7 world-wide round trips per second
* 2,000 round trips per second within a data center

#### Latency numbers visualized

![](https://camo.githubusercontent.com/77f72259e1eb58596b564d1ad823af1853bc60a3/687474703a2f2f692e696d6775722e636f6d2f6b307431652e706e67)

#### Source(s) and further reading

* [Latency numbers every programmer should know - 1](https://gist.github.com/jboner/2841832)
* [Latency numbers every programmer should know - 2](https://gist.github.com/hellerbarde/2843375)
* [Designs, lessons, and advice from building large distributed systems](http://www.cs.cornell.edu/projects/ladis2009/talks/dean-keynote-ladis2009.pdf)
* [Software Engineering Advice from Building Large-Scale Distributed Systems](https://static.googleusercontent.com/media/research.google.com/en//people/jeff/stanford-295-talk.pdf)

## Company engineering blogs

* [Airbnb Engineering](http://nerds.airbnb.com/)
* [Atlassian Developers](https://developer.atlassian.com/blog/)
* [Autodesk Engineering](http://cloudengineering.autodesk.com/blog/)
* [AWS Blog](https://aws.amazon.com/blogs/aws/)
* [Cloudera Developer Blog](http://blog.cloudera.com/blog/)
* [Dropbox Tech Blog](https://tech.dropbox.com/)
* [Engineering at Quora](http://engineering.quora.com/)
* [Ebay Tech Blog](http://www.ebaytechblog.com/)
* [Evernote Tech Blog](https://blog.evernote.com/tech/)
* [Facebook Engineering](https://www.facebook.com/Engineering)
* [Flickr Code](http://code.flickr.net/)
* [Foursquare Engineering Blog](http://engineering.foursquare.com/)
* [GitHub Engineering Blog](http://githubengineering.com/)
* [Google Research Blog](http://googleresearch.blogspot.com/)
* [Groupon Engineering Blog](https://engineering.groupon.com/)
* [Heroku Engineering Blog](https://engineering.heroku.com/)
* [Hubspot Engineering Blog](http://product.hubspot.com/blog/topic/engineering)
* [Instagram Engineering](http://instagram-engineering.tumblr.com/)
* [LinkedIn Engineering](http://engineering.linkedin.com/blog)
* [Microsoft Engineering](https://engineering.microsoft.com/)
* [Netflix Tech Blog](http://techblog.netflix.com/)
* [Paypal Developer Blog](https://devblog.paypal.com/category/engineering/)
* [Pinterest Engineering Blog](http://engineering.pinterest.com/)
* [Quora Engineering](https://engineering.quora.com/)
* [Reddit Blog](http://www.redditblog.com/)
* [Salesforce Engineering Blog](https://developer.salesforce.com/blogs/engineering/)
* [Slack Engineering Blog](https://slack.engineering/)
* [Spotify Labs](https://labs.spotify.com/)
* [Twilio Engineering Blog](http://www.twilio.com/engineering)
* [Twitter Engineering](https://engineering.twitter.com/)
* [Uber Engineering Blog](http://eng.uber.com/)
* [Yahoo Engineering Blog](http://yahooeng.tumblr.com/)
* [Yelp Engineering Blog](http://engineeringblog.yelp.com/)

## Credits and resources

* [Hired in tech](http://www.hiredintech.com/system-design/the-system-design-process/)
* [donnemartin/system-design-primer](https://github.com/donnemartin/system-design-primer)
* [shashank88/system_design](https://github.com/shashank88/system_design)
* [checkcheckzz/system-design-interview](https://github.com/checkcheckzz/system-design-interview)
* [Crack the Sytem Design Interview](http://www.puncsky.com/blog/2016/02/14/crack-the-system-design-interview/)
* [FreemanZhang/system-design](https://github.com/FreemanZhang/system-design)
* [Success in Tech Youtube channel](https://www.youtube.com/channel/UC-vYrOAmtrx9sBzJAf3x_xw)
* [How to ace system design interview](https://www.palantir.com/2011/10/how-to-ace-a-systems-design-interview/)
* [David Malan Scalability Web Development lecture](https://youtu.be/-W9F__D3oY4)
* [Scalability for Dummies](http://www.lecloud.net/tagged/scalability)
* [Database sharding](http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html)
