# gRPC

## What is gRPC?

gRPC is a high-performance framework for communication between services.

It is commonly used for **service-to-service communication** in distributed systems and microservices.

```text
Service A
    |
    | gRPC
    ↓
Service B
```

## Key Characteristics

* High-performance communication
* Commonly used for internal service-to-service communication
* Uses **Protocol Buffers (Protobuf)** for defining services and messages
* Supports strongly typed contracts
* Supports streaming
* Uses HTTP/2

## Why Use gRPC?

Compared with traditional REST/JSON communication, gRPC can provide:

* Efficient binary serialization
* Strongly typed contracts
* Efficient communication
* Streaming support
* Code generation from `.proto` definitions

## Basic Flow

```text
Client
   |
   | gRPC Request
   ↓
gRPC Server
   |
   ↓
Response
```

## Where is gRPC Useful?

Common use cases:

* Microservices communication
* Internal APIs
* High-performance service-to-service communication
* Streaming applications

## gRPC vs REST

| Feature         | gRPC                                 | REST                               |
| --------------- | ------------------------------------ | ---------------------------------- |
| Common protocol | HTTP/2                               | HTTP/1.1 or HTTP/2                 |
| Data format     | Protobuf                             | Usually JSON                       |
| Contract        | Strongly typed `.proto`              | Usually OpenAPI/schema             |
| Streaming       | Built-in support                     | Possible, but different mechanisms |
| Browser usage   | Requires gRPC-Web / suitable gateway | Very common                        |
| Typical use     | Internal service communication       | Public APIs / web APIs             |

## Key Takeaway

```text
gRPC
  ↓
HTTP/2
  ↓
Protocol Buffers
  ↓
Strongly typed communication
  ↓
Fast service-to-service communication
```
