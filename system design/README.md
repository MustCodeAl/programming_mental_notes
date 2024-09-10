##  Scaling your software:


1. **Establish Design Scope** - Think about specific featuers to build, How fast will users adopt it, and leveraging existing services to simplify the design
2. **Propose high-level design** - come up with initial blueprint for the design, draw box diagrams with key components, and do back-of-the-envelope calcuations
3. **Design deep dive** - Agreed on the overall goals and feature scope, obtain feedback, and Had some initial ideas about areas to focus on 
4. **Identify the System Bottlenecks** and **potential improvements** - Look at error cases, operation issues, and handle next scale curve

Plan out: anticipated scales in 3 months, 6 months, and a year
Look for: existing services you might leverage to simplify the design
Draw out: clients (mobile/web), APIs, web servers, data stores, cache, CDN, message queue, etc.
Discuss: server failures, network loss, etc., monitor metrics and error logs, System roll outs

* Understand the requirements of the problem.
* Suggest multiple approaches if possible.
* Design the most critical components first.
* Don’t jump into a solution without clarifying the requirements and assumptions.
* Ask for help and Communicate. Don't think in silence.



### iterative process.
* Keep web tier stateless
* Build redundancy at every tier
* Cache data as much as you can
* Support multiple data centers
* Host static assets in CDN
* Scale your data tier by sharding
* Split tiers into individual services
* Monitor your system and use automation too

### initial guidelines
* Sends requests to websites public ip
* load balance, forwards request to appropriate server
* Have servers uses private ips
* decouple different components of the system
* server should retrieves state from cache,
* cache does write requests to master databases and
* cache does read requests to database slave reeplicas
* use message queues in cache for better decoupled components
* have workers handle the queue
* have logging, mettrics and automation


#### Definitinos
A message queue is a durable component, stored in memory, that supports asynchronous communication. It serves as a buffer and distributes asynchronous requests. The basic architecture of a message queue is simple. Input services, called producers/publishers, create messages, and publish them to a message queue. Other services or servers, called consumers/subscribers, connect to the queue, and perform actions defined by the messages. The model is shown in Figure 1-17.
Logging: Monitoring error logs is important because it helps to identify errors and problems in the system. You can monitor error logs at per server level or use tools to aggregate them to a centralized service for easy search and viewing.

Metrics: Collecting different types of metrics help us to gain business insights and understand the health status of the system. Some of the following metrics are useful:
• Host level metrics: CPU, Memory, disk I/O, etc.
• Aggregated level metrics: for example, the performance of the entire database tier, cache tier, etc.
• Key business metrics: daily active users, retention, revenue, etc.

Automation: When a system gets big and complex, we need to build or leverage automation tools to improve productivity. Continuous integration is a good practice, in which each code check-in is verified through automation, allowing teams to detect problems early. Besides, automating your build, test, deploy process, etc. could improve developer productivity significantly

#### Optimization Tips
* Memory is fast but the disk is slow. Avoid disk seeks if possible.
* Simple compression algorithms that are fast. Compress data before sending it over the internet if possible.
* Data centers are usually in different regions, and it takes time to send data between them. So estimate queries per second and storage requirements


##### Estimating Storage requirements
• Rounding and Approximation. It is difficult to perform complicated math operations during the interview. For example, what is the result of “99987 / 9.1”? There is no need to spend valuable time to solve complicated math problems. Precision is not expected. Use round numbers and approximation to your advantage. The division question can be simplified as follows: “100,000 / 10”.
• Write down your assumptions. It is a good idea to write down your assumptions to be referenced later.
• Label your units. When you write down “5”, does it mean 5 KB or 5 MB? You might confuse yourself with this. Write down the units because “5 MB” helps to remove ambiguity.
• Commonly asked back-of-the-envelope estimations: QPS, peak QPS, storage, cache, number of servers, etc. 
