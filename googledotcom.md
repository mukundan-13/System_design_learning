# What Happens When You Type `google.com` and Press Enter

## 1. URL Processing

1. Type `google.com` and press **Enter**.
2. Browser validates the URL format.
3. Browser adds the appropriate scheme, such as `https://`.

---

## 2. DNS Resolution

### Browser / OS DNS Cache

1. Browser checks its **DNS cache** for the IP address of `google.com`.
2. If not found, the **OS DNS cache** is checked.
3. If still not found, the request is sent to a **DNS resolver**, such as:
   - ISP DNS resolver
   - `8.8.8.8` — Google Public DNS
   - `1.1.1.1` — Cloudflare DNS

### DNS Resolution Process

1. Resolver checks its own cache.
2. If there is a cache miss, it queries the **Root DNS servers**.
3. Root server responds with the appropriate **`.com` TLD (Top-Level Domain) server**.
4. Resolver queries the `.com` TLD server.
5. TLD server returns the **authoritative name server (NS)** for `google.com`.
6. Resolver queries Google's authoritative DNS server.
7. Authoritative server returns the IP address.
8. DNS response can be cached at multiple levels according to its TTL.

---

## 3. TCP Connection

Once the browser has the IP address:

1. Browser initiates a TCP connection to the server, typically on **port 443** for HTTPS.
2. **SYN** → sent to server.
3. **SYN + ACK** → received from server.
4. **ACK** → sent back to server.
5. **TCP connection established.**

### TCP 3-Way Handshake

```text
Client                         Server
  |                              |
  | -------- SYN --------------> |
  |                              |
  | <----- SYN + ACK ----------- |
  |                              |
  | -------- ACK --------------> |
  |                              |
  |       Connection Ready       |


## 4. TLS Handshake

**TLS = Transport Layer Security**

1. Browser sends **TLS ClientHello**.
2. Server responds with **ServerHello** and its TLS certificate.
3. Browser validates the certificate.
4. Both sides establish the cryptographic keys required for the secure session.
5. **Secure encrypted channel is ready.**

> Note: SSL is the older predecessor of TLS. Modern HTTPS uses TLS.

---

## 5. HTTP Request

1. Browser sends an **HTTP request** over the secure connection.
2. Request reaches Google's server infrastructure.
3. The request may pass through load balancers, edge services, CDN/cache layers, etc.

### Example

```http
GET / HTTP/2
Host: google.com


# 6. Server Infrastructure

A simplified request flow:

```text
Browser
   ↓
Load Balancer / Edge
   ↓
CDN / Cache
   ↓
Backend Service
   ↓
Database / Other Services
   ↓
Response
```

1. **Load Balancer / Edge** routes the request.
2. **CDN / Cache** may be checked.
3. If content is cached → it can be served immediately.
4. If not cached → the request is forwarded to backend services.
5. Backend services fetch or generate the required data.
6. Server produces the HTTP response.

> The exact infrastructure and request path can vary depending on the service, location, protocol, and request.

---

# 7. Response Back to You

The server sends an HTTP response, for example:

```http
HTTP/2 200 OK
```

The response travels through the TLS-encrypted connection.

Data travels through:

```text
Server
   ↓
Routers
   ↓
ISP / Network
   ↓
Your Local Network
   ↓
Your Device
   ↓
Browser
```

The browser receives the data.

TCP provides:

* Reliable delivery
* Ordered delivery
* Reassembly of data into the correct order

> With HTTP/3, the transport layer uses QUIC instead of TCP.

---

# 8. Browser Rendering

After receiving the response, the browser starts processing and rendering the page.

## HTML → DOM

```text
HTML
  ↓
HTML Parser
  ↓
DOM Tree
```

The browser parses the HTML and creates the **DOM (Document Object Model)** tree.

## CSS → CSSOM

```text
CSS
  ↓
CSS Parser
  ↓
CSSOM
```

The browser parses CSS and creates the **CSSOM (CSS Object Model)**.

## Rendering Process

The browser:

1. Decrypts the TLS-protected response.
2. Parses HTML → creates the DOM tree.
3. Parses CSS → creates the CSSOM.
4. Requests JavaScript files and other required resources.
5. Additional resources may trigger further network requests:

   * Images
   * Fonts
   * CSS
   * JavaScript
   * Favicon
6. Combines DOM + CSS information.
7. Calculates element sizes and positions during **Layout**.
8. Draws visual content during **Paint**.
9. Combines layers during **Compositing**, often with GPU assistance.
10. Final pixels appear on the screen.

## Rendering Pipeline

```text
HTML
  ↓
DOM
  ↓
       CSS
        ↓
      CSSOM
        ↓
   DOM + CSSOM
        ↓
    Render Tree
        ↓
      Layout
        ↓
       Paint
        ↓
    Compositing
        ↓
      Screen
```

---

# 9. Optimization Layer

Modern browsers and web protocols optimize this process using several techniques.

## Browser Cache

Stores frequently used resources locally on the device.

Examples:

* CSS
* JavaScript
* Images
* Fonts

This can prevent the browser from downloading the same resources repeatedly.

---

## HTTP/2

HTTP/2 allows multiple requests and responses to be multiplexed over a single TCP connection.

Key features include:

* Multiplexing
* Connection reuse
* Header compression
* Stream prioritization

Instead of requiring a separate connection for every request, multiple streams can share the same connection.

---

## HTTP/3

HTTP/3 uses **QUIC** as its transport protocol instead of TCP.

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
```

QUIC can reduce connection setup overhead and improve performance on some networks.

---

## CDN Caching

A **CDN (Content Delivery Network)** stores frequently requested content at geographically distributed locations.

```text
User
  ↓
Nearby CDN Edge
  ↓
Cached Content
```

This can reduce latency because content may be served from a location closer to the user.

---

## Server-Side Caching

Server-side caching reduces repeated backend or database work.

Example:

```text
User Request
     ↓
Backend
     ↓
Cache Check
     ↓
 ┌───────────────┐
 │ Cache Hit?    │
 └───────────────┘
    ↓        ↓
   Yes       No
    ↓        ↓
Return     Database
Cache        ↓
Data       Store in Cache
             ↓
          Return Data
```

---

# Complete Flow

A simplified example of what happens when you type `google.com` into a browser:

```text
Type google.com
       ↓
Browser validates URL
       ↓
DNS Cache Check
       ↓
OS DNS Cache Check
       ↓
DNS Resolver
       ↓
Root DNS
       ↓
.com TLD
       ↓
Google Authoritative DNS
       ↓
IP Address
       ↓
TCP 3-Way Handshake
       ↓
TLS Handshake
       ↓
Encrypted Connection
       ↓
HTTP Request
       ↓
Load Balancer / Edge
       ↓
CDN / Cache
       ↓
Backend Services
       ↓
HTTP Response
       ↓
TLS
       ↓
Internet
       ↓
Browser
       ↓
HTML → DOM
CSS → CSSOM
       ↓
Render Tree
       ↓
Layout
       ↓
Paint
       ↓
Compositing
       ↓
🖥️ Web Page Appears
```

