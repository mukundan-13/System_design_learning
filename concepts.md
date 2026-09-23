# Microservices

## 1. Cascade Failure

A **cascade failure** occurs when failure in one service causes failures in other dependent services.

Example:

```text
Service A
   |
   ↓
Service B
   |
   ↓
Service C
```

Suppose Service C becomes slow or unavailable.

```text
Service A → Service B → Service C
                       ❌
```

Service B may keep waiting for Service C.

Then Service A keeps waiting for Service B.

Eventually, resources such as:

* Threads
* Connections
* Memory
* CPU

can become exhausted.

The failure can propagate through the system.

```text
Service C fails
      ↓
Service B becomes slow
      ↓
Service A becomes slow
      ↓
More requests accumulate
      ↓
System degradation
```

---

# 2. Circuit Breaker

A **Circuit Breaker** helps prevent failures from propagating through dependent services.

Instead of continuously calling an unhealthy service, the circuit breaker temporarily stops calls to it.

```text
Service A
    |
    v
Circuit Breaker
    |
    v
Service B
```

The Circuit Breaker has **3 states**:

```text
       failure threshold
Closed ─────────────────→ Open
  ↑                         |
  |                         |
  | success                 | timeout/wait
  |                         ↓
  └──────────────────── Half-Open
```

## Closed

Normal operation.

Requests are allowed through.

```text
Client → Circuit Breaker → Service
```

Failures are monitored.

If failures cross a configured threshold:

```text
Closed → Open
```

---

## Open

Requests are not sent to the failing service.

```text
Client
  |
  v
Circuit Breaker
  |
  X
Service
```

The application can immediately return an error or fallback response instead of repeatedly calling the unhealthy service.

This helps prevent unnecessary load on the failing service and reduces the risk of cascading failures.

---

## Half-Open

After waiting for a configured period, the circuit breaker allows a limited number of test requests.

```text
Circuit Breaker
      |
      | Test request
      ↓
   Service
```

If the service is healthy:

```text
Half-Open → Closed
```

If the service is still failing:

```text
Half-Open → Open
```

---

# 3. API Gateway

An **API Gateway** acts as an entry point between clients and backend services.

```text
                 API Gateway
                      |
       +--------------+--------------+
       |              |              |
       ↓              ↓              ↓
   User Service   Order Service   Payment Service
```

## Responsibilities

An API Gateway can handle:

* Authentication
* Authorization
* Routing
* Load balancing
* Rate limiting
* Request/response transformation
* Logging
* TLS termination
* Aggregation

### Example

```text
GET /users
     |
     ↓
API Gateway
     |
     ↓
User Service
```

```text
GET /orders
     |
     ↓
API Gateway
     |
     ↓
Order Service
```

The client does not necessarily need to know the internal service structure.

---

# 4. API Composition

API Composition combines data from multiple services into a single response.

Without API composition:

```text
Client
  |
  +--> User Service
  |
  +--> Order Service
  |
  +--> Payment Service
```

The client makes multiple requests.

With API composition:

```text
Client
   |
   v
API Gateway / Composer
   |
   +--> User Service
   |
   +--> Order Service
   |
   +--> Payment Service
   |
   v
Combined Response
```

Example:

```text
GET /user-dashboard
```

The composer may retrieve:

```text
User details
+
Orders
+
Payment information
```

and return them as one response.

### Benefit

Reduces the number of calls the client needs to make.

### Trade-off

The composition layer becomes dependent on multiple services and can add latency.

---

# 5. CQRS

**CQRS = Command Query Responsibility Segregation**

The idea is to separate:

* **Commands** — change data
* **Queries** — read data

Instead of using the same model for both:

```text
             Application
              /       \
           Write      Read
             \         /
              Database
```

CQRS can separate the read and write models.

```text
                Application
                 /       \
                /         \
          Commands       Queries
             |              |
             ↓              ↓
       Write Model      Read Model
             |              |
             ↓              ↓
       Write DB         Read DB
```

## Command

Changes state.

Examples:

```text
Create Order
Update User
Delete Product
```

## Query

Reads data.

Examples:

```text
Get User
Get Orders
Get Product
```

### Why use CQRS?

It can be useful when:

* Read and write workloads are very different
* Read models need different structures
* Independent scaling is useful
* Complex read requirements exist

### Important

CQRS does **not necessarily mean** using two separate databases. The key idea is the separation of command and query responsibilities.

---

# 6. Distributed Tracing

In microservices, a single user request can travel through many services.

```text
Client
  |
  ↓
API Gateway
  |
  ↓
User Service
  |
  ↓
Order Service
  |
  ↓
Payment Service
```

If the request takes 5 seconds, it can be difficult to identify where the delay happened.

Distributed tracing helps track the request across services.

---

# 7. Trace and Span

A **trace** represents the complete journey of a request.

A **span** represents one operation within that trace.

```text
Trace
│
├── API Gateway span
│
├── User Service span
│
├── Order Service span
│
└── Payment Service span
```

Example:

```text
Total request = 2 seconds

API Gateway  = 100 ms
User Service = 200 ms
Order Service = 300 ms
Payment      = 1400 ms
```

The trace makes it easier to identify that the Payment Service consumed most of the time.

---

# 8. OpenTelemetry

**OpenTelemetry (OTel)** is a standard/tooling ecosystem for collecting telemetry data.

It supports:

* Traces
* Metrics
* Logs

For distributed tracing, applications can instrument requests and generate trace/span information.

```text
Services
   |
   ↓
OpenTelemetry
   |
   ↓
Telemetry Backend
```

---

# 9. Zipkin

**Zipkin** is a distributed tracing system.

A simplified architecture:

```text
Microservices
     |
     ↓
Tracing Data
     |
     ↓
Zipkin
     |
     ↓
Trace Visualization
```

You can inspect a request and see how it traveled across services.

---

# 10. How These Concepts Connect

These concepts are not isolated.

A typical microservices architecture can look like:

```text
                         Client
                           |
                           ↓
                     API Gateway
                           |
             +-------------+-------------+
             |             |             |
             ↓             ↓             ↓
        User Service  Order Service  Payment Service
             |             |             |
             +-------------+-------------+
                           |
                     Service Calls
                           |
                    Circuit Breaker
                           |
                    Failure Protection
```

Meanwhile:

```text
API Gateway
     |
     ↓
OpenTelemetry
     |
     ↓
Distributed Trace
     |
     ↓
Zipkin / tracing backend
```

And for data:

```text
Commands ──→ Write Model
Queries  ──→ Read Model

        = CQRS
```

---

# Quick Revision

| Concept             | Main Purpose                                   |
| ------------------- | ---------------------------------------------- |
| gRPC                | Efficient service-to-service communication     |
| Cascade Failure     | Failure propagating between dependent services |
| Circuit Breaker     | Prevent repeated calls to failing services     |
| API Gateway         | Single entry point for client requests         |
| API Composition     | Combine responses from multiple services       |
| CQRS                | Separate command and query responsibilities    |
| Distributed Tracing | Track requests across microservices            |
| OpenTelemetry       | Collect traces, metrics and logs               |
| Zipkin              | Distributed tracing system                     |

---

# Mental Model

```text
                 Client
                    |
                    ↓
              API Gateway
                    |
          +---------+---------+
          |         |         |
          ↓         ↓         ↓
       Service A Service B Service C
          |         |         |
          +---------+---------+
                    |
              Circuit Breaker
                    |
              Failure Protection


Request Flow
     ↓
OpenTelemetry
     ↓
Distributed Trace
     ↓
Zipkin / Tracing Backend


Data
     ↓
Commands → Write Model
Queries  → Read Model
     ↓
    CQRS
```
