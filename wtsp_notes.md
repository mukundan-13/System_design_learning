# WhatsApp Messaging System — System Design Notes

## 1. Signal Protocol

WhatsApp uses the **Signal Protocol** to provide end-to-end encryption.

The main security properties include:

### Forward Secrecy

Forward secrecy ensures that if a current encryption key is compromised, previously exchanged messages cannot easily be decrypted.

The system continuously uses new keys so that old messages are protected by older keys.

### Key Rotation

Encryption keys are regularly changed/rotated during communication.

This limits the amount of data that can be compromised if a particular key is exposed.

### Protection Against Replay Attacks

A replay attack happens when an attacker captures a previously valid message and tries to send it again.

The messaging protocol uses mechanisms such as message keys, counters/nonces, and authenticated encryption to ensure that old messages cannot simply be replayed as new messages.

---

# 2. Pull Model vs Push Model

## Pull Model

In the pull model, the client periodically asks the server:

```text
Client → Server
        "Do I have any new messages?"
```

The server responds with available messages.

### Example

```text
Client
   ↓
"Any new messages?"
   ↓
Server
   ↓
"Yes, here are 2 messages"
```

This approach can result in unnecessary requests when there are no new messages.

---

## Push Model

In the push model, the server informs the client when something needs to be delivered.

```text
Server → Client
         "You have a new message"
```

Messaging applications commonly maintain a persistent connection so that the server can deliver events to the client quickly.

---

# 3. WhatsApp Persistent Connection

WhatsApp maintains a **long-lived connection** between the mobile device and its servers.

A simplified representation:

```text
Mobile Phone
     │
     │ Long-lived TCP connection
     │
     ▼
WhatsApp Server
```

The persistent connection allows messages and other events to be delivered without repeatedly creating a new connection.

Historically, WhatsApp has used **TCP-based persistent connections** for messaging.

---

# 4. How WhatsApp Scales to Millions of Users

A messaging system needs to handle a very large number of concurrent users and connections.

Some important techniques include:

### 4.1 Erlang

WhatsApp has historically used **Erlang** extensively for its server-side messaging infrastructure.

Erlang is designed for highly concurrent and fault-tolerant systems.

It is particularly useful for workloads involving:

* Large numbers of concurrent connections
* Message processing
* Fault tolerance
* Distributed systems

---

## 4.2 Stateless Frontend Servers

Frontend/application servers can be designed to remain **stateless**.

Instead of storing user-specific session information locally:

```text
             ┌── Server 1
Client ──────┼── Server 2
             └── Server 3
```

Any suitable server can handle a request.

State can instead be stored in distributed databases, caches, or other backend systems.

This makes horizontal scaling easier.

---

## 4.3 Data Sharding

A huge amount of user and message data cannot efficiently be stored on a single database server.

The data can be divided into multiple **shards**.

```text
                 Database
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Shard 1     Shard 2     Shard 3
     Users A-H   Users I-P   Users Q-Z
```

Each shard handles a subset of the data.

This allows the system to distribute storage and database workload across multiple machines.

---

## 4.4 Distributed Message Queues

Message queues can be used to handle asynchronous message processing.

```text
Sender
   │
   ▼
Message Service
   │
   ▼
Distributed Queue
   │
   ▼
Recipient Service
   │
   ▼
Receiver
```

Queues are particularly useful when the recipient is temporarily offline.

The message can remain queued until it can be delivered.

---

# 5. Image and Media Handling

Large media files such as images and videos should not necessarily travel through the same path as normal text messages.

A simplified flow is:

```text
User A
  │
  │ Upload image
  ▼
Media Server / Storage
  │
  │ Returns media reference
  ▼
User A
  │
  │ Sends encrypted media reference
  ▼
WhatsApp Server
  │
  ▼
User B
```

The message can contain information that allows the recipient to retrieve the media.

The actual media can be stored separately from the messaging infrastructure.

---

# 6. Video Encoding

Video files are large and expensive to transmit.

WhatsApp can perform **media processing/encoding on the device or through its media infrastructure**, depending on the operation and platform.

A simplified flow is:

```text
Original Video
      │
      ▼
Encoding / Compression
      │
      ▼
Smaller Video
      │
      ▼
Upload
```

Encoding/compression reduces:

* File size
* Upload bandwidth
* Download bandwidth
* Storage requirements

---

# 7. Mobile Push Notifications

When a user is not actively connected to the application, the operating system's push-notification infrastructure can be used to wake or notify the device.

## Android

Android applications commonly use:

**Firebase Cloud Messaging (FCM)**

```text
WhatsApp Server
      │
      ▼
     FCM
      │
      ▼
Android Device
```

---

## iOS

iOS applications use Apple's push notification infrastructure:

**Apple Push Notification service (APNs)**

```text
WhatsApp Server
      │
      ▼
     APNs
      │
      ▼
   iPhone
```

The push notification can notify the device that new content is available, after which the application can establish/use its connection to retrieve the relevant data.

---

# 8. WhatsApp Message Delivery Flow

A simplified messaging architecture can be divided into several layers.

```text
Sender Phone
     │
     ▼
┌─────────────────────┐
│ Encryption Layer    │
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│ Persistent          │
│ Connection          │
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│ Routing Server      │
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│ Message Queue       │
└─────────────────────┘
     │
     ▼
Recipient Phone
     │
     ▼
┌─────────────────────┐
│ Decryption Layer    │
└─────────────────────┘
```

---

# 9. Step-by-Step Message Flow

Suppose **User A sends a message to User B**.

### Step 1 — Encryption

The message is encrypted on User A's device.

```text
"Hello"
   │
   ▼
Encrypted Message
```

The message is protected using the end-to-end encryption protocol.

---

### Step 2 — Persistent Connection

The encrypted message is sent through the connection between User A's device and the messaging infrastructure.

```text
User A
  │
  │ Persistent connection
  ▼
WhatsApp Infrastructure
```

---

### Step 3 — Routing

The messaging infrastructure determines where the message needs to go.

```text
User A
  │
  ▼
Routing Service
  │
  ▼
User B
```

The routing layer is responsible for getting the message to the correct recipient.

---

### Step 4 — Queueing

If User B is offline, the message can be temporarily stored in a server-side queue.

```text
User A
  │
  ▼
Server
  │
  ▼
Message Queue
  │
  │ User B offline
  │
  └───────────────┐
                  │
              User B
              comes online
                  │
                  ▼
             Message delivered
```

Once User B becomes available, the queued message can be delivered.

---

### Step 5 — Decryption

The recipient's device receives the encrypted message and uses its cryptographic keys to decrypt it.

```text
Encrypted Message
       │
       ▼
Recipient Device
       │
       ▼
   Decryption
       │
       ▼
     "Hello"
```

The server does not simply receive the plaintext and forward it to the recipient.

---

# 10. WhatsApp Tick System

The check marks provide an indication of the message's delivery state.

## Single Tick ✓

The message has been successfully sent to WhatsApp's server/infrastructure.

```text
Sender
  │
  ▼
WhatsApp Server

✓
```

It does **not** necessarily mean that the recipient's device has received the message.

---

## Double Tick ✓✓

The message has been delivered to the recipient's device.

```text
Sender
  │
  ▼
Server
  │
  ▼
Recipient Device

✓✓
```

This means the recipient device has acknowledged receipt.

---

## Blue Tick ✓✓

The message has been **read/seen**, when read receipts are enabled and applicable.

```text
Sender
  │
  ▼
Recipient Device
  │
  ▼
Message opened/read

✓✓ Blue
```

It is important to distinguish **decryption** from **reading**:

> A message can be decrypted by the recipient's device without necessarily meaning that the user has opened/read it.

---

# 11. Overall Architecture

A simplified high-level architecture looks like this:

```text
                    ┌─────────────────────┐
                    │   WhatsApp Client   │
                    │      User A         │
                    └──────────┬──────────┘
                               │
                         Encryption
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Persistent TCP      │
                    │ Connection          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Routing / Messaging │
                    │ Servers             │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
             ┌──────────────┐     ┌──────────────┐
             │ Message      │     │ Data /       │
             │ Queues       │     │ Sharded DB   │
             └──────┬───────┘     └──────────────┘
                    │
                    │
                    ▼
            ┌──────────────────┐
            │ Recipient Device │
            │     User B       │
            └────────┬─────────┘
                     │
                 Decryption
                     │
                     ▼
                  Message
```

---

# 12. Key Components to Remember

For a system-design interview, remember these major components:

| Component                      | Purpose                                                                |
| ------------------------------ | ---------------------------------------------------------------------- |
| **Signal Protocol**            | End-to-end encryption                                                  |
| **Forward Secrecy**            | Protects previously exchanged messages if current keys are compromised |
| **Key Rotation**               | Regularly changes cryptographic keys                                   |
| **Persistent TCP Connection**  | Enables continuous communication between device and server             |
| **Routing Service**            | Determines where messages should be delivered                          |
| **Message Queue**              | Temporarily stores messages when recipients are unavailable            |
| **Data Sharding**              | Distributes large datasets across multiple servers                     |
| **Distributed Infrastructure** | Handles massive numbers of users/connections                           |
| **FCM**                        | Push notification infrastructure for Android                           |
| **APNs**                       | Push notification infrastructure for iOS                               |
| **Media Storage**              | Handles large images/videos separately from normal messages            |
| **Encryption/Decryption**      | Protects message contents end-to-end                                   |

---

# 13. Simplified Mental Model

When designing a WhatsApp-like messaging system, think:

```text
             SEND
              │
              ▼
         Encrypt Message
              │
              ▼
      Persistent Connection
              │
              ▼
        Routing Service
              │
       ┌──────┴──────┐
       │             │
    Online         Offline
       │             │
       ▼             ▼
   Deliver       Message Queue
       │             │
       │       User comes online
       │             │
       └──────┬──────┘
              ▼
       Recipient Device
              │
              ▼
          Decrypt
              │
              ▼
           Display
```

The core idea is:

**Encryption → Connection → Routing → Queueing → Delivery → Decryption → Read/ACK**
