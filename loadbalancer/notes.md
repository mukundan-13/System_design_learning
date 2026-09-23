# Load Balancing

## Types of Load Balancer

1. **Hardware**
2. **Software**

   * NGINX
   * HAProxy (High Availability Proxy)
   * ELB — Elastic Load Balancer
3. **DNS**

---

## Traffic Types

### East-West Traffic

* East-west traffic load balancing

### Global-Level Load Balancing

* Sends requests to the nearest/appropriate data center
* Achieved using **GeoDNS**

---

## OSI Model Used in Load Balancing

1. **Physical Layer**

2. **Data Link Layer**

3. **Network Layer**

4. **Transport Layer (L4)**

   * Handles **TCP and UDP**
   * Takes source IP, destination IP, and port information
   * Sends the request to the respective port
   * Used when simplicity and better performance are required

5. **Session Layer**

6. **Presentation Layer**

7. **Application Layer (L7)**

   * Watches:

     * Headers
     * Request type
     * Cookies
     * Request body
   * Makes decisions about where to send the request, such as:

     * Application server
     * CDN
   * Slower than L4 but more intelligent

---

## AWS Load Balancers

AWS has two important types of load balancers:

1. **NLB — Network Load Balancer**

   * Network/Transport-level load balancing
   * L4

2. **ALB — Application Load Balancer**

   * Application-level load balancing
   * L7

---

# Load Balancing Algorithms

## 1. Round Robin

Requests are distributed sequentially between servers.

```text
A → B → C → A → B → C
```

Each server gets requests in rotation.

### Weighted Round Robin

Servers are assigned different weights based on their capacity.

For example:

```text
Server A → Weight 3
Server B → Weight 2
Server C → Weight 1
```

The more powerful server receives more requests.

---

## 2. Least Connection & Weighted Least Connection

The request is sent to the server with the fewest active connections.

Useful for **long-lived connections**, such as:

* WebSockets
* Long-lived connections
* Persistent connections

### Weighted Least Connection

Considers both:

* Number of active connections
* Server capacity/weight

---

## 3. IP Hashing

```text
Source IP
    ↓
Hash
    ↓
Server
```

The source IP is hashed and mapped to a server.

Future requests from the same source can be sent to the same server.

### Used For

* Stateful sessions
* Session affinity

### Drawback

What happens if that server fails?

The request may need to be remapped to another server.

---

## 4. Consistent Hashing

Used in distributed systems such as:

* Memcached clusters
* Redis clusters
* CDN edge routing

Cassandra also uses consistent-hashing-related partitioning concepts.

When scaling horizontally, consistent hashing helps reduce unnecessary remapping and cache misses.

---

## 5. Random

The load balancer randomly selects a server.

Can be used as a simple/fallback strategy depending on the system.

---

## 6. Least Response Time

The load balancer considers server response time.

```text
Faster server → More requests
Slower server → Fewer requests
```

The server responding faster is more likely to receive the next request.

---

# Health Checks

## 1. Passive Check

The load balancer observes normal traffic and detects failures.

For example:

```text
Server → 500 Error
```

If a server repeatedly fails, the load balancer can stop sending new requests to it.

---

## 2. Active Check

The load balancer actively sends a request to a health endpoint.

Examples:

```text
/health
/ping
```

Example:

```text
LB → /health → Server
```

If the server returns:

```text
200 OK
```

the server is considered healthy.

If the health check fails repeatedly, the load balancer can stop sending traffic to that server.

Once the server becomes healthy again, traffic can be sent to it again.

---

# Sticky Sessions

If a user goes to Server B:

```text
User → LB → Server B
```

future requests from that user can continue going to Server B.

Sticky sessions can be implemented using:

* Cookie-based affinity
* IP-based affinity

### Problem

The user becomes dependent on a particular server.

```text
User → Server B
          ↓
       Server fails
```

---

# Avoiding Sticky Sessions

To avoid depending on a particular server, externalize session state.

For example, use **Redis**:

```text
                ┌──→ Server A ──┐
                │               │
User → Load Balancer → Server B ├──→ Redis
                │               │
                └──→ Server C ──┘
```

Now any application server can access the user's session.

Benefits:

* Fault tolerant
* Horizontally scalable
* Less dependency on a specific server
* Avoids the need for sticky sessions

---

# SSL/TLS

## SSL/TLS Termination

The load balancer terminates the TLS connection.

```text
Client
   │
 HTTPS
   │
   ↓
Load Balancer
   │
   │ Decrypts TLS
   ↓
Backend Server
```

Because the load balancer terminates TLS, it can inspect HTTP-level information and perform L7 routing.

---

## SSL/TLS Passthrough

The load balancer does **not** terminate TLS.

```text
Client
   │
Encrypted TLS
   │
   ↓
Load Balancer
   │
Encrypted TLS
   │
   ↓
Backend Server
```

The backend server handles TLS termination/decryption.

---

# System Design Questions

## 1. Design a URL Shortener for 1 Billion Users

Things to think about:

* Load balancing
* Horizontal scaling
* Database
* Caching
* Database partitioning/sharding
* Replication
* URL generation
* Rate limiting
* Availability

---

## 2. If There Is a Sudden 10x Spike in Traffic, How Will You Handle It?

I will use **horizontal scaling** through **autoscaling groups**.

Load balancers will distribute traffic across the available application servers.

```text
              Load Balancer
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Server A    Server B    Server C
```

Autoscaling can add more servers when traffic increases.

---

## 3. How Will You Manage Sessions in Distributed Systems?

Externalize state using **Redis** and avoid sticky sessions.

```text
Server A ──┐
Server B ──┼──→ Redis
Server C ──┘
```

This allows requests to be handled by different application servers.

---

## 4. Design a Rate Limiter

Things to learn:

* Fixed Window
* Sliding Window
* Token Bucket
* Leaky Bucket
* Redis-based distributed rate limiting

---

## 5. What If the Load Balancer Fails?

Deploy the load balancer in **High Availability**.

Things to learn:

* DNS failover
* Floating IP / Virtual IP
* Managed load-balancing services
* Multiple load balancer instances
* Health checks
* Failover mechanisms

---

# Topics to Learn Further

* DNS Failover
* Floating IP / Virtual IP
* Managed Load Balancers
* High Availability
* Rate Limiting
* Autoscaling
* Consistent Hashing
* Sticky Sessions
* TLS Termination
* L4 vs L7 Load Balancing



# Load Balancing — System Design Notes

## 1. What is Load Balancing?

A **Load Balancer (LB)** distributes incoming traffic across multiple servers/instances.

Instead of:

```text
Client
   |
   v
Server
```

we use:

```text
             +--> Server A
Client --> LB+--> Server B
             +--> Server C
```

### Why do we need it?

* Distribute traffic across servers
* Prevent a single server from becoming overloaded
* Improve availability
* Support horizontal scaling
* Detect unhealthy servers
* Enable failover
* Route requests intelligently

---

# 2. Types of Load Balancers

## A. Hardware Load Balancer

Physical devices dedicated to load balancing.

```text
Clients
   |
   v
[Hardware LB]
   |
   +--> Server A
   +--> Server B
   +--> Server C
```

**Advantages**

* Very high performance
* Specialized hardware
* Can handle huge traffic

**Disadvantages**

* Expensive
* Hardware capacity is limited
* Scaling requires additional hardware

---

## B. Software Load Balancer

Load balancing is implemented using software.

Examples:

* NGINX
* HAProxy
* AWS Elastic Load Balancing (managed service)

Software load balancers are easier to scale and configure compared with dedicated hardware appliances.

---

## C. DNS Load Balancing

DNS can return different IP addresses depending on the request.

For example:

```text
example.com
     |
     v
   DNS
  /   \
 LB-1  LB-2
```

DNS can also be used for **global traffic management**.

---

# 3. East-West vs North-South Traffic

### North-South Traffic

Traffic entering or leaving the system.

```text
Internet
   |
   v
Load Balancer
   |
   v
Application Servers
```

Example:

A user accessing your web application.

### East-West Traffic

Traffic between services inside a system/data center.

```text
Service A
    |
    v
Service B
    |
    v
Service C
```

This is common in microservice architectures.

Load balancing can happen between internal services as well.

---

# 4. Global Load Balancing

Global load balancing distributes users across multiple data centers or regions.

Example:

```text
                Global LB
                    |
        +-----------+-----------+
        |                       |
     US DC                   India DC
        |                       |
   Servers                  Servers
```

The goal can be to route a user toward an appropriate region based on factors such as:

* Geographic location
* Network latency
* Data-center health
* Traffic load

### GeoDNS

**GeoDNS** can return different IP addresses based on the geographic location of the DNS requester.

Example:

```text
User from India
      |
      v
    DNS
      |
      v
India Data Center
```

```text
User from USA
      |
      v
    DNS
      |
      v
USA Data Center
```

> GeoDNS is one approach to global traffic management. Other approaches include Anycast and dedicated global load-balancing services.

---

# 5. OSI Model and Load Balancing

Load balancing can operate at different layers of the OSI model.

```text
Layer 7  Application
Layer 6  Presentation
Layer 5  Session
Layer 4  Transport
Layer 3  Network
Layer 2  Data Link
Layer 1  Physical
```

---

# 6. Layer 4 Load Balancing

**L4 = Transport Layer**

It primarily deals with:

* TCP
* UDP

The load balancer can make routing decisions using information such as:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
```

Example:

```text
Client
192.168.1.10:5000
       |
       v
    L4 LB
       |
       +----> Server A:443
       |
       +----> Server B:443
```

### Advantages

* Fast
* Lower processing overhead
* Can handle very high traffic
* Does not need to understand application-level HTTP details

### Disadvantage

It has less application-level information available for routing decisions.

---

# 7. Layer 7 Load Balancing

**L7 = Application Layer**

An L7 load balancer can understand application-level protocols such as HTTP/HTTPS.

It can inspect things such as:

* HTTP headers
* HTTP method
* URL/path
* Cookies
* Hostname
* Query parameters
* Sometimes request body, depending on the implementation

Example:

```text
/api/users/*     --> User Service

/api/orders/*    --> Order Service

/static/*        --> CDN / Static Server
```

### Advantages

More intelligent routing.

Example:

```text
GET /users
      |
      v
User Service

GET /orders
      |
      v
Order Service
```

### Disadvantage

More processing is required compared with L4, so it can introduce additional overhead.

---

# 8. L4 vs L7

| Feature              | L4                                 | L7                        |
| -------------------- | ---------------------------------- | ------------------------- |
| Layer                | Transport                          | Application               |
| Common protocols     | TCP/UDP                            | HTTP/HTTPS                |
| URL inspection       | No                                 | Yes                       |
| HTTP headers         | No                                 | Yes                       |
| Cookies              | No                                 | Yes                       |
| Routing intelligence | Lower                              | Higher                    |
| Processing overhead  | Lower                              | Higher                    |
| Typical use          | High-performance transport routing | Application-aware routing |

---

# 9. AWS Load Balancers

AWS Elastic Load Balancing provides different load-balancing options.

Two important ones are:

### Network Load Balancer (NLB)

Primarily operates at **Layer 4**.

Used when high-performance TCP/UDP/TLS traffic handling is required.

### Application Load Balancer (ALB)

Operates at **Layer 7**.

Useful for HTTP/HTTPS applications and content-based routing.

Example:

```text
                 ALB
                  |
       +----------+----------+
       |                     |
 /users/*                /orders/*
       |                     |
 User Service           Order Service
```

---

# 10. Load Balancing Algorithms

## 10.1 Round Robin

Requests are distributed sequentially.

```text
Request 1 --> A
Request 2 --> B
Request 3 --> C
Request 4 --> A
Request 5 --> B
Request 6 --> C
```

Pattern:

```text
A -> B -> C -> A -> B -> C
```

### Problem

It assumes servers can handle approximately similar workloads.

For example:

```text
Server A = powerful
Server B = medium
Server C = weak
```

Sending an equal number of requests to all three may not be optimal.

Also, **Round Robin does not wait for a request to complete before sending the next request**. That was an important correction to your original note.

---

# 11. Weighted Round Robin

Servers receive different weights.

Example:

```text
Server A = weight 5
Server B = weight 3
Server C = weight 1
```

Traffic is distributed approximately according to those weights.

```text
A A A A A B B B C
```

Useful when servers have different capacities.

---

# 12. Least Connections

The load balancer sends the new request to the server with the fewest active connections.

Example:

```text
Server A -> 10 connections
Server B -> 4 connections
Server C -> 7 connections
```

Next request:

```text
              |
              v
              B
```

Useful when connections have significantly different lifetimes.

Common examples:

* WebSocket connections
* Long-lived TCP connections
* Some persistent application connections

---

# 13. Weighted Least Connections

Combines:

* Number of active connections
* Server capacity/weight

A more powerful server can handle more connections.

---

# 14. IP Hashing

The client IP is hashed to determine the server.

```text
Client IP
   |
   v
Hash Function
   |
   v
Server B
```

Future requests from that same source IP can be mapped to the same server.

Useful when you want a degree of session affinity.

### Problem

If the selected server fails, requests may be remapped.

Another issue is that many users can appear behind the same NAT/public IP, which can create uneven distribution.

---

# 15. Consistent Hashing

Consistent hashing reduces the amount of remapping when nodes are added or removed.

Conceptually:

```text
             Server A
                |
       ---------------------
      /                     \
   Key1                    Key2
      \                     /
       ---------------------
                |
             Server B
```

It is commonly useful in distributed systems such as:

* Distributed caches
* Redis clusters
* Memcached-style caching architectures
* CDN routing
* Distributed storage systems

### Why?

Suppose:

```text
Before:

A B C
```

A key is mapped to B.

If we add D using simple modulo hashing:

```text
hash(key) % number_of_servers
```

many keys may move to different servers.

This causes many cache misses.

Consistent hashing reduces the number of keys that need to move.

> Cassandra uses consistent-hashing-related partitioning concepts, but its partitioning implementation is more specific than simply saying "Cassandra is based only on consistent hashing."

---

# 16. Random

The load balancer randomly selects a server.

```text
Request
   |
   +--> Random --> Server B
```

Simple, but usually less predictable than algorithms that consider server load.

Random selection can sometimes be useful in simple systems or as part of more sophisticated strategies.

---

# 17. Least Response Time

The load balancer considers server response performance and routes traffic toward servers responding faster.

Example:

```text
Server A -> 50 ms
Server B -> 200 ms
Server C -> 80 ms
```

A new request is more likely to be directed toward a faster server, depending on the LB's implementation.

This can be useful when server performance varies significantly.

---

# 18. Health Checks

A load balancer needs to know whether a backend server is healthy.

There are two common approaches.

## Active Health Check

The load balancer actively sends a health-check request.

Example:

```text
LB
 |
 +--> GET /health --> Server A
 |
 +--> GET /health --> Server B
 |
 +--> GET /health --> Server C
```

If:

```text
Server B --> 200 OK
```

LB considers it healthy.

If it repeatedly fails:

```text
Server B --> timeout / 500
```

LB can temporarily remove it from the pool.

When it becomes healthy again, it can be added back.

---

## Passive Health Check

The load balancer observes real traffic.

For example:

```text
Client --> LB --> Server B

Server B repeatedly returns errors
```

The load balancer can detect repeated failures and stop routing traffic to that server.

No separate health-check request is necessarily required.

---

# 19. Sticky Sessions

Sticky sessions mean that requests from a client are consistently routed to the same backend server.

Example:

```text
User A
  |
  v
  LB
  |
  v
Server B

User A's future requests
  |
  v
  LB
  |
  v
Server B
```

Common approaches include:

### Cookie-based affinity

The load balancer uses a cookie to maintain affinity.

### IP-based affinity

The client's IP is used to determine the backend.

---

# 20. Problem With Sticky Sessions

Consider:

```text
             LB
          /      \
         /        \
      Server A   Server B

Users --> Server A
```

If Server A becomes overloaded:

```text
Server A = 95% CPU
Server B = 20% CPU
```

sticky sessions may continue sending those users to Server A.

This reduces the effectiveness of load distribution.

If Server A fails, users may also lose their in-memory session state.

---

# 21. Avoiding Sticky Sessions

A common approach is to **externalize session state**.

Instead of:

```text
Server A
  |
  +--> Session stored in memory
```

use:

```text
             +--> Server A --+
             |               |
User --> LB -+--> Server B --+--> Redis
             |               |
             +--> Server C --+
```

All application servers can access the shared session store.

This makes the application more suitable for horizontal scaling and improves failover behavior.

Common technologies:

* Redis
* Database
* Distributed session stores

---

# 22. SSL Termination

HTTPS traffic is encrypted.

With **SSL/TLS termination**, the load balancer decrypts the incoming TLS connection.

```text
Client
   |
 HTTPS
   |
   v
  LB
   |
 decrypts TLS
   |
 HTTP/HTTPS
   |
   v
Backend
```

The important point is that once the LB terminates TLS, it can inspect HTTP-level information and perform L7 routing.

Example:

```text
/api/users  --> User Service
/api/orders --> Order Service
```

### TLS Passthrough

The load balancer forwards the encrypted TLS connection to the backend without terminating TLS.

```text
Client
   |
 encrypted TLS
   |
   v
  LB
   |
 encrypted TLS
   |
   v
Backend
```

The backend server performs the TLS termination/decryption.

> TLS passthrough itself is not an "L4-only requirement." It is a way of forwarding encrypted traffic without terminating TLS at the load balancer.

---

# 23. Common System Design Questions

## Q1. Design a URL Shortener for 1 Billion Users

Important areas to think about:

```text
Client
  |
  v
Load Balancer
  |
  v
Application Servers
  |
  +--------> Cache
  |
  +--------> Database
```

Things to consider:

* URL generation
* Database design
* Read/write ratio
* Caching
* Horizontal scaling
* Load balancing
* Database partitioning/sharding
* Replication
* Rate limiting
* Availability
* Analytics
* Expiration of URLs

---

# 24. What if Traffic Suddenly Increases 10x?

Example:

```text
Normal traffic
    |
    v
100 requests/sec

Sudden spike
    |
    v
1000 requests/sec
```

A possible architecture:

```text
                  Load Balancer
                       |
          +------------+------------+
          |            |            |
       Server A     Server B     Server C
```

Use **horizontal scaling** to add more application instances.

An autoscaling mechanism can increase/decrease the number of instances based on metrics such as:

* CPU
* Request count
* Latency
* Queue length
* Custom application metrics

The load balancer distributes traffic across the available healthy instances.

Also consider:

* Caching
* CDN
* Database scaling
* Queues for asynchronous work
* Rate limiting
* Backpressure
* Autoscaling limits

> Simply adding servers does not solve every 10x spike. The database, cache, queue, network, or another downstream dependency can become the bottleneck.

---

# 25. How Do You Manage Sessions in a Distributed System?

Avoid storing important session state only inside one application server.

Instead:

```text
             +--> Server A --+
             |               |
Client --> LB+--> Server B --+--> Redis
             |               |
             +--> Server C --+
```

Session state is stored in a shared external system.

This allows requests to reach different servers.

### Goal

```text
Request 1 --> Server A
Request 2 --> Server C
Request 3 --> Server B
```

All servers can still retrieve the user's session.

This reduces the need for sticky sessions.

---

# 26. Design a Rate Limiter

A rate limiter controls how many requests a client can make during a period.

Example:

```text
User
 |
 | 100 requests/minute
 v
Rate Limiter
 |
 +---- allowed --> Application
 |
 +---- exceeded --> 429 Too Many Requests
```

Common algorithms:

* Fixed Window
* Sliding Window
* Token Bucket
* Leaky Bucket

For a distributed system, a shared store such as Redis can be used so that multiple application servers share the same rate-limit state.

---

# 27. What If the Load Balancer Fails?

The load balancer itself can become a single point of failure if deployed incorrectly.

Instead, deploy multiple load-balancer instances or use a highly available managed load-balancing service.

Conceptually:

```text
                 DNS
                  |
          +-------+-------+
          |               |
       LB-1             LB-2
          |               |
          +-------+-------+
                  |
          Application Servers
```

Common techniques include:

### High Availability

Multiple LB instances are deployed so that failure of one does not bring down the service.

### DNS Failover

DNS can redirect traffic when an endpoint becomes unavailable, depending on the DNS/service configuration.

### Floating / Virtual IP

A virtual IP can move between redundant nodes in some architectures.

### Managed Load Balancers

Cloud providers can provide highly available load-balancing infrastructure.

---

# 28. Important Mental Model

When designing a distributed application, think about load balancing at different levels.

```text
                    Internet
                       |
                       v
                Global Traffic
                   Routing
                       |
                       v
                Load Balancer
                  L4 / L7
                       |
          +------------+------------+
          |            |            |
       Server A     Server B     Server C
          |            |            |
          +------------+------------+
                       |
                 Shared State
                       |
                    Redis
                       |
                    Database
```

Then ask:

1. How is traffic distributed?
2. What happens when a server fails?
3. How do we detect unhealthy servers?
4. Do we need session affinity?
5. Can we externalize state?
6. How do we scale horizontally?
7. What happens during a traffic spike?
8. What happens if the load balancer itself fails?
9. Where does TLS terminate?
10. Is routing happening at L4 or L7?

---

# 29. Quick Revision

### Load Balancer Types

```text
Hardware
Software
DNS / Global Traffic Management
```

### Important Layers

```text
L4 -> TCP / UDP -> fast, less application awareness

L7 -> HTTP / HTTPS -> intelligent application-aware routing
```

### Algorithms

```text
Round Robin
Weighted Round Robin
Least Connections
Weighted Least Connections
IP Hash
Consistent Hashing
Random
Least Response Time
```

### Health Checks

```text
Active -> LB sends health-check requests

Passive -> LB observes real traffic/errors
```

### Sessions

```text
Sticky Session
     |
     v
Can create uneven distribution

Externalized Session
     |
     v
Redis / shared store
     |
     v
Better horizontal scalability
```

### TLS

```text
TLS Termination
Client --> LB --> Backend

TLS Passthrough
Client --> LB --> Backend
          encrypted traffic remains encrypted
```

### High Availability

```text
Multiple LB instances
+
Health checks
+
Failover
+
Redundancy
```

---

# 30. Key Takeaway

A load balancer is not simply:

> "Something that sends requests to different servers."

It is an important part of a distributed system that helps with:

**traffic distribution + health detection + scalability + availability + intelligent routing + failover.**
