## Table Of Contents

* [What is system design?](#what-is-system-design)
* [Basic things you should know](#basic-things-you-should-know)
* [Step by step how to approach a system design interview question](#step-by-step)

## Basic things you should know

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

### Constraints and use cases

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
