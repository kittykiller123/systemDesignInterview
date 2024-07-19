# single server setup


![[Pasted image 20240715220624.png]]


*request flow and traffic source*

![[Pasted image 20240715221040.png]]

> Domain Name System (DNS)

The traffic to your web server comes from two
sources: 
- web application
- mobile application.

## database

> With the growth of the user base, one server is not enough, and we need multiple servers: one for web/mobile traffic, the other for the database (Figure 1-  3). Separating web/mobile traffic (web tier) and database (data tier) servers allows them to be scaled independently.

![[Pasted image 20240715221304.png]]

### which DB to use

#### relational databases
*SQL DB*

Relational databases represent and store data in tables and rows.

#### non-relational databases
*NoSQL DB*
These databases are grouped into
four categories: key-value stores, graph stores, column stores, and document stores. Join
operations are generally not supported in non-relational databases.

适用于：
- 应用需要很低延迟
- 数据是非结构性的/没有关系型数据
- 只需要序列化/反序列化数据
- 需要存储大量的数据


## Vertical scaling vs horizontal scaling

### vertical scaling
缩放，增加更多power（CPU,RAM...)
*当traffic is low时较好的选择--简单*
- 但是有严重局限性
	- 无法在一台服务器中添加无限的CPU和内存
### Horizontal scaling
扩展（增加更多服务器）

在先前的例子中，用户直接连接到web server。

**scenario：**

1. web server is offline
2. many users access web server simultaneously(同时地) and it reaches the web server's load limit
	- *load balancer!*

## load balancer
> A load balancer evenly distributes incoming traffic among web servers that are defined in a load-balanced set


![how a load balancer works](figure1-4.png)

> - 用户直接连接到负载均衡器的公共IP
> - 私有ip用于server之间的通信
> - A private IP is an IP address reachable only between servers in the same network; 


figure1-4中，我们加入了第二台web server和load balancer，提高了web tier的availability。

1. 如果网站流量增长迅速，而两台服务器不足以处理流量，负载均衡器则可以正常处理，只需要需要添加更多服务器到 Web 服务器池中，负载均衡器会自动开始向它们发送请求。
2. 如果server 1 go offline，所有流量会被路由到server2

***what about data tier？***

_只有一个database，不支持failover和redundancy_

## database replication

> Database replication can be used in many database management systems, usually with a master/slave relationship between the original (master) and the copies(slaves)


![[Pasted image 20240716202940.png]]

*notes:*

1. a master db only supports write operations
2. a slave database only supports read operations
3. all the data-modifying commands must be sent to the master
4. most applications require a much higher ratio of reads to writes-> *the number of slave usually larger than the number of master*

#### advantages:

- better performance：允许并行处理更多query
- reliability:数据会在多个位置复制
- high availability：即使一个db offline，也可以访问在另一个db server中的数据
![[Pasted image 20240716203632.png]]


![[Pasted image 20240716203800.png]]


**是时候来改进load/response time！**

- 可以通过添加`cache layer`和`shifting static content`(js/css/img/video files) to the [[CDN(Content delivery network)]]


## cache

> A cache is a temporary storage area that stores the result of expensive responses or frequently accessed data in memory so that subsequent requests are served more quickly. As illustrated in Figure 1-6, every time a new web page loads, one or more database calls are executed to fetch data. The application performance is greatly affected by calling the database repeatedly. 
> The cache can mitigate（缓和） this problem.


## cache tier

![[Pasted image 20240716204205.png]]


### considerations for using cache

- ***when to use cache***
- ***expiration policy***
- ***consistency***
- ***mitigating failures***
	- *SPOF*
	- ![[Pasted image 20240717140206.png | 400]]
- ***eviction policy***: LRU, [[LFU]],FIFO



## Content delivery network(CDN)

> [!NOTE]
> A CDN is a network of geographically dispersed servers used to deliver static content. CDN servers cache static content like images, videos, CSS, JavaScript files, etc.


**Dynamic content caching**: material[9]

![[Pasted image 20240717140451.png | 500]]



![[Pasted image 20240717140506.png | 550]]

### considerations of using a CDN

- Cost
- Setting an appropriate cache expiry
- CDN fallback: 应用程序如何处理CDN失败？
- Invalidating files
	- using APIs provided by CDN vendors(供应商)
	- 使用对象版本控制

![[Pasted image 20240717140818.png | 600]]

1. Static assets (JS, CSS, images, etc.,) are no longer served by web servers. They are fetched from the CDN for better performance.
2. The database load is lightened by caching data.


## Stateless web tier
现在轮到考虑水平缩放web层——我们需要将状态（例如user session data用户会话数据）从web层中移动出来。*A good practice is to store session data in the persistent storage such as relational database or NoSQL.  Each web server in the cluster can access state data from databases. This is called stateless web tier.*


## stateful architecture

**stateful server VS. stateless server**

- 前者从一个请求到下一个请求都会记住客户端数据（状态）
- 后者不保存任何状态信息

![[Pasted image 20240717141551.png | 550]]

在上图中，用户A的会话数据和profile image都存储在server1中。要对用户A进行身份验证，必须将HTTP请求路由到server1.如果将请求发送到其他服务器（如服务器 2），则身份验证将失败，因为服务器 2 不包含用户 A 的会话数据。

***issue：***
![[Pasted image 20240717142746.png]]

> [!NOTE]
> Sticky Sessions 是一种在负载均衡中使用的会话保持技术，又称为会话粘性或会话绑定。它的作用是确保同一个用户的所有请求都被路由到同一个后端服务器。这在需要保持用户会话状态的应用中非常有用，比如电商网站或在线游戏。------FROM GPT4-o

***sticky sessions:***

- **会话保持**：当一个用户首次访问应用时，负载均衡器会将用户的请求分配给一台后端服务器，并在之后的请求中继续将该用户的请求路由到同一台服务器。
    
- **实现方法**：常见的实现 Sticky Sessions 的方法包括：
    
    - **Cookie**：负载均衡器在用户首次访问时设置一个 cookie，标记分配的服务器，在后续请求中根据该 cookie 将请求路由到同一服务器。
    - **IP 地址**：根据用户的 IP 地址进行会话绑定，但这种方法在使用代理服务器或共享 IP 地址时效果不佳。



## Stateless architecture
![[Pasted image 20240717143016.png | 550]]


_来自用户的http请求可能被发送到任何web server上！这些服务器从共享数据存储中获取状态数据。_

- simpler，more robust，more scalable!


![[Pasted image 20240717143300.png | 550]]


- 在上图中，我们将session data从web层中移出，并将其持久性存储（可以是relational db、redis、NoSQL等，这里选择NoSQL是因为它易于scale）。
- *Autoscaling：* adding or removing web servers automatically based on the traffic load. 

After the state data is removed out of web servers, auto-scaling of the web tier is easily achieved by adding or removing servers based on traffic load.

> [!ASSUMPTION]
> 你的网站发展迅速，并吸引了大量的国际用户。为了提高可用性并在更广泛的地理区域内提供更好的用户体验，支持多个数据中心至关重要。


## Data centers

![[Pasted image 20240717144109.png | 550]]

上图展示了一个具有两个数据中心的示例设置。

在发生任何重大的data center中断时，我们将所有流量引导到一个健康的data center

![[Pasted image 20240717144410.png | 550]]

### 一些技术问题需要被解决！

- ***Traffic redirection:*** GeoDNS
- ***Data synchronization:*** 
- ***Test and deployment:***


为了进一步扩展我们的系统，需要解耦(decoupling)系统的不同组件，以便它们可以独立地缩放。


## Message queue

- a durable component
- stored in memory
- supports asynchronous communication

![[Pasted image 20240717153903.png | 550]]

building a scalable and reliable application!

- 当consumer不可用时，producer可以将消息发布到队列
- 当producer不可用，consumer也可以从队列中读取信息


![[Pasted image 20240717155428.png | 550]]

> [!use case]
>  your application supports photo customization, including cropping, sharpening, blurring, etc. Those customization tasks take time to complete. In Figure 1-18, web servers publish photo processing jobs to the message queue. Photo processing workers pick up jobs from the message queue and asynchronously perform photo customization tasks. The producer and the consumer can be scaled independently. When the size of the queue becomes large, more workers are added to reduce the processing time. However, if the queue is empty most of the time, the number of workers can be reduced.


## Logging, metrics, automation

我们地网站已经发展为服务于一个大型企业，投资于logging、metrics、automation是至关重要的。

#### Logging





#### Metrics
收集不同类型指标可以帮助我们获得业务简介和了解系统地健康状况。如:
- host level metric: CPU, Memory, disk I/O, etc
- Aggregated(聚合) level metrics: the performance of the entire database tier, cache tier, etc
- key business metrics: daily active users, retention, revenue, etc(日活跃用户、留存率、收入等)



#### automation

当一个系统变得大而复杂时，我们需要构建或利用自动化工具来提高生产率。持续集成是一种很好的实践，其中每个代码签入都通过自动化进行验证，使团队能够及早发现问题。此外，自动化您的构建、测试、部署过程等等。可以显著提高开发人员的生产力。


### Adding message queues and different tools



![[Pasted image 20240717163156.png | 550]]

随着数据量增长，我们的数据库负载越来越大······

It is time to scale the data tier！

## Database scaling

### Vertical scaling
> Vertical scaling, also known as scaling up, is the scaling by adding more power (CPU, RAM, DISK, etc.) to an existing machine.

#### drawbacks:
- cost much
- hardware limits
- greater risk of SPOF




### Horizontal scaling
Horizontal scaling, also known as *sharding* , is the practice of adding more servers. Figure 1-20 compares vertical scaling with horizontal scaling.

![[Pasted image 20240717164037.png | 550]]

![[Pasted image 20240717165933.png | 550]]

                        一个shared database的示例

![[Pasted image 20240717170030.png | 550]]


在实施sharding strategy时最重要的因素是sharding key的选择（选择一颗可以均匀分布数据的key）。
如图1-22， userid为sharding key

#### new challenges and complexities:

- ***resharding data***: -> consistent hashing
- ***celebrity problem***: 热点关键问题，对特定shard的过度访问可能导致server overload->we may need to allocate a shard for each celebrity.
- ***join and de-normalization:*** Once a database has been sharded across multiple servers, it ishard to perform join operations across database shards. A common workaround is to de-normalize the database so that queries can be performed in a single table.

![[Pasted image 20240717170556.png | 550]]


## Millions of users and beyond
*summary:*

• Keep web tier stateless
• Build redundancy at every tier
• Cache data as much as you can
• Support multiple data centers
• Host static assets in CDN
• Scale your data tier by sharding
• Split tiers into individual services
• Monitor your system and use automation tools


# reference materials

[1] Hypertext Transfer Protocol: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[2] Should you go Beyond Relational Databases?:
https://blog.teamtreehouse.com/should-you-go-beyond-relational-databases
[3] Replication:  https://en.wikipedia.org/wiki/Replication_(computing)
[4] Multi-master replication:
https://en.wikipedia.org/wiki/Multi-master_replication
[5] NDB Cluster Replication: Multi-Master and Circular Replication:
https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster-replication-multi-master.html
[6] Caching Strategies and How to Choose the Right One:
https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/
[7] R. Nishtala, "Facebook, Scaling Memcache at," 10th USENIX Symposium on Networked
Systems Design and Implementation (NSDI ’13).
[8] Single point of failure: https://en.wikipedia.org/wiki/Single_point_of_failure
[9] Amazon CloudFront Dynamic Content Delivery:
https://aws.amazon.com/cloudfront/dynamic-content/
[10] Configure Sticky Sessions for Your Classic Load Balancer:
https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html
[11] Active-Active for Multi-Regional Resiliency:
https://netflixtechblog.com/active-active-for-multi-regional-resiliency-c47719f6685b
[12] Amazon EC2 High Memory Instances:
https://aws.amazon.com/ec2/instance-types/high-memory/
[13] What it takes to run Stack Overflow:
http://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow
[14] What The Heck Are You Actually Using NoSQL For:
http://highscalability.com/blog/2010/12/6/what-the-heck-are-you-actually-using-nosql-
for.html