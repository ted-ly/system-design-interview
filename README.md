## Table Of Contents

* [What is system design?](#what-is-system-design)
* [Basic things you should know](#basic-things-you-should-know)
* [Step by step how to approach a system design interview question](#step-by-step-how-to-approach-a-system-design-interview-question)

## What is system design?

The process of defining the architecture, modules, interfaces, and data for a system to satisfy specified requirements. [System design](https://en.wikipedia.org/wiki/Systems_design) could be seen as the application of systems theory to product environment.


## Basic things you should know

* [Availability patterns](#availability-patterns)
* [Domain name system](#domain-name-system)
* [Content delivery network](#content-delivery-network)
* [Consistent hashing](#consistent-hashing)
* [CAP theorem](#cap-theorem)
* [Load balancing](#load-balancing)
* [Caching](#caching)
* [Sharding or data paritioning](#sharding-or-data-partitioning)
* [Indexes](#indexes)
* [Proxies](#proxies)
* [Queues](#queues)
* [Redundancy and replication](#redundancy-and-replication)
* [SQL and noSQL](#sql-and-nosql)
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

## Credits & Resources

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
