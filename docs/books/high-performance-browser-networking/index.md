# High Performance Browser Networking

## Chapter 1 - Primer on Latency and Bandwidth

`Latency`

- Time from source sending a packet to the destination receiving it

`Bandwidth`

- Maximum throughput of a logical or a physical communication path

`CDN`

- Distributing content around the globe, serving content from nearby location to the client.

`Last-Mile Latency`

- It's often that the last few stops of the packet is the slowest. To connect your home/device/office to internet, your ISP needs to route the cables throughout the neighbourhood, aggregate the signal, and forward it to local routing node.

Use traceroute to check the hops to the destination.

```shell
traceroute google.com
```

### Awareness

Speed of light places a hard limit on how fast energy can travel (300.000 km/s). Current fiber optic lines already can process packets in nearly 200.000 km/s.

So, even reaching the speed of light would bring just 30-40% improvement.

Thus, we must improve performance of our applications by architecting and optimizing our protocols and networking code while keeping in mind the limitations of available bandwidth, and the speed of light.

- Reduce roundtrips
- Move the data closer to client
- Build applications that hide latency through caching, pre-fetching

## Chapter 2 - Building Blocks of TCP

`IP`

- Internet protocol, provides host-to-host routing and addressing

`TCP`

- Transmission Control Protocol, provides reliable network running on unreliable channel. Most used protocol. You are guaranteed that all bytes sent, will be equal to bytes received, in the sent order.

TCP is optimized to accurate delivery rather than speed, thus, it may bring challanges when it comes to optimizing for web performance.

### Three-way handshake

All TCP connections starts with 3 way handshake. Once the handshake is complete, data can flow between client and server. Client can send a packet right after the ACK (last packet of the 3 step handshake), and the server `must` wait for ACK before dispatching any data.

This means that each connection will have a roundtrip of latency before any data can be transferred.

This delay by 3-way handshake makes new TCP connections expensieve, thus, connection reuse is critical for optimizing.

#### Flow control

Mechanism to prevent the sender from overwhelming the receiver with data that it won't be able to process.

To address this, each side of TCP connection advertises its own receive window(`rwnd` value) with ACK packet.

#### Slow start algorithm

TCP slow start is an algorithm which balances the speed of a network connection. Slow start gradually increases the amount of data transmitted until it finds the network’s maximum carrying capacity.

- The sender’s initial packet contains a small congestion window, which is determined based on the sender’s maximum window.

- The receiver acknowledges the packet and responds with its own window size. If the receiver fails to respond, the sender knows not to continue sending data.

- Sender increases the next packet’s window size. The window size gradually increases until the receiver can no longer acknowledge each packet, or until either the sender or the receiver’s window limit is reached.

- Once a limit has been determined, slow start’s job is done. Other congestion control algorithms take over to maintain the speed of the connection.

### Bandwidth-delay product

Product of data link's capacity and its end to end delay. Result is the maximum amount of unacknowledged data that can be in flight at any point in time.

How big should flow control(rwnd) and congestion control(cwnd) window values be?

`Example`

> Assuming minimum of cwnd and rwnd is 16KB and roundtrip time is 100ms

16KB == 131,072 bits

131,072 bits / 0.1s = 1,310,720 bits/s

1,310,720 / 1,000,000 = `1,31` Mbps

Regardless of available bandwidth, TCP connection will not exceed a 1,31Mbps data rate! To achieve a better data rate, we would need to increase the minimum window size or reduce the roundtrip.

### TCP / UPD - When to use what

TCP brings features to the table such as reliable delivery and `in-order` packet transmission, which is great but comes at the expense of extra latency.

However, not all applications' requirements are same, thus, not all would need reliable delivery and can handle packet loss without a sweat.

These can be video streaming, 3D games, audio streaming, where small losses of packets would not affect the overall experience, however, extra latency would.

This is also why WebRTC uses UDP as its base transport.

### Summary

`TCP`

- 3-way handshake introduces full round trip latency

- Slow-start is applied to each new connection

- Flow and congestion controls regulate the throughput of all connections

- Throughput is regulated by current congestion window size

#### Things to think about

`Increasing TCP's Initial Congestion Window`

Larger starting congestion window allows TCP transfers more data in the first roundtrip and can significantly accelerates the window growth - thus, can be critical for short lived connections.

`Slow-start restart`

Disabling slow-start after idle will improve performance of long lived TCP connections.

`Window Scaling`

Enabling window scaling increases the maximum receive window size and allows high-latency connections to achieve better throughput.

`TCP Fast Open`

Read more...

#### Performance checklist

- upgrade kernel
  
- ensure `cwnd` size is set to 10

- disable slow-start after idle

- ensure that window scaling is enabled

- transfer the minimum data

- compress transferred data

- position servers close to the client

- reuse established TCP connections when possible

## Chapter 3 - Building blocks of UDP

User Datagram Protocol is a minimalistic transport protocol.

It's known by the features that it omits.

`No guarantee of message delivery`

`No guarantee of order of delivery`

`No connection state tracking`

`No congestion control`

It's mainly used as starting point to build up other protocols and frameworks.

It's simple, stateless protocol which leaves almost all design decisions to application/framework developer. Example framework that uses UDP under the hood is WebRTC.

If you want to leverage UDP for your own application, make sure to research and read about it's limitations. Recommendations for applications:

- Must tolerate wide range of Internet path conditions
- `should` control the rate of transmission
- `should perform` congestion control
- `should` use bandwidth similar to TCP
- `should` back off retransmission counters following a loss
- `should` not send datagrams that exceed path MTU
- `should` handle datagram loss, duplication, and reordering
- `should` be robust to delivery delays up to 2 mins
- `should` enable IPv4, and must `enable` IPv6 checksum
- `may` use keepalives when needed(15 seconds intervals)

## Chapter 4 - Transport Layer Security (TLS)

- SSL protocol, initially developed @Netscape.

- It was implemented right on top of TCP, enabling protocols above it such as HTTP, SMTP to have an encrypted communication.

- During standartization of SSL by IETF, it was renamed to TLS, thus naming used interchangeably, but technically they refer to a different version of the protocol.

### Goals

- Encryption

- Authentication

- Data Integrity

### TLS Handshake

Before data can start flowing over TLS, encrypted tunnel must be negotiated. Client and server should agree on a version of TLS, choose a ciphersuite, and verify certificates if necessary. Each of these steps require a rountrip, thus adding `startup latency` to all TLS connections.

Considering an example where 28ms is one way delay between client and server.

- 0ms We wait for TCP handshake to finish, which introduces a full roundtrip

- 56ms Client sends specs in plain text such as TLS protocol version, list of ciphersuites, and other TLS options

- 84ms Server picks the TLS protocol version, decides on ciphersuite from the list, attaches its certificate and sends response back to client.

- 112ms Assuming both sides are able to negotiate on common version and cipher and happy with the certificate provided by the server, client initiates either RSA or the Diffie-Hellman key exchange

- 140ms Server processes the key exchange parameters, checks the message integrity by verifying MAC and returns encrypted "Finished" message back to client

- 168ms Client decrypts the message, verifies the MAC and if all is well, tunnel is established and application data can now be sent.

> Now the data start flowing between client and server

Optimizations should be used alonside such as TLS False Start and TLS Session Resumption in order to avoid having handshake roundtrips on each connection.

### Key exchange algorithms

- RSA

Traditional symmetric key exchange. Slowly being phased out due to vulnerabilities that comes from theft of private key by an attacker.

Read more [here](https://en.wikipedia.org/wiki/RSA_(cryptosystem))

- Diffie-Hellman 

Enables forward secrecy by generation of keys on each exchange. Thus having access to private key on server does not make previous or further exchanges of data vulnerable.

Read more [here](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)

> Public-key cryptography is only used during setup of the TLS tunnel. Further communication uses symmetric-key cryptography due to performance reasons.

### Server Name Indication (SNI)

Encrypted TLS tunnel can be established between any two TCP peers. Only thing client needs to know is the IP address of the server.

What happens when we want to host multiple websites on the same IP address but with different TLS certificates. It doesn't work.

To address this problem, Server Name Indication extension was introduced to TLS so that client can indicate the HOSTNAME at the start of the handshake. In this case, Web Server can inspect the SNI hostname and select appropriate certificate to continue the handshake.

### TLS Record Protocol

First 5 bytes -> Headers(Content-Type, Version, Length)

5..n bytes -> Payload

n..m bytes -> MAC

m..p bytes -> Padding(block cyphers only)

Maximum TLS Record Size is 16KB.

To decrypt/encrypt the record, entire record must be available, thus, picking the right record size is important for optimization.

Optimizations:

- TLS False Start

- Early Termination(CDN, close servers)

- Session caching and stateless resumption

### TLS Compression

Disable, because, it has vulnerabilities and does not differentiate between content types during compression and recompresses already compressed data.

### HTTP Strict Transport Security

Is a security policy mechanism that allows server to declare access rules to a compliant browser via simple HTTP header:

- Strict-Transport-Security: max-age=31536000

It instructs user agent to enforce:

- All requests to the origin should be sent over HTTPs
  
- All insecure links and client requests should be converted to HTTPs
  
- In case of certificate error, message is displayed to user and it's not allowed to circumvent the warning.

- max-age specifies the lifetime of the specified HSTS ruleset in seconds(in this case 365 days)

Performance-wise it eliminates need to HTTP-HTTPs redirection and delegates this task over to client.

### Performance Chechklist

- Optimize for TCP

- Upgrade TLS libraries

- Enable & configure session caching and stateless resumption

- Configure forward secrecy ciphers to enable TLS False Start.

- Terminate TLS sessions closer to client

- Use dynamic TLS record sizing to optimize for latency and throughput

- Ensure the certificate size does not overflow the initial congestion window to reduce unnecessary rountrips

- Configure OCSP stapling

- Disable TLS compression

- Configure SNI support

- Append HTTP Strict Transport Security header

## Chapter 9 - Brief History of HTTP

HTTP 0.9 by Tim Berners-Lee. Initial version only consisted of METHOD and PATH which returned simple hypertext document with no headers or metadata.

Features of HTTP 0.9:

- Client-server, request-response protocol

- ASCII protocol, running over TCP/IP link

- Designed to transfer hypertext documents(HTML)

- Connection between client and server is terminated after each request

NGNIX/Apache still supports 0.9 because there isn't much to it.

HTTP 1.0 brought wide adoption and features.

- Request/Response headers and metadata
  
- Multiple lines
  
- Response is not limited to hypertext

- Connection between server and client closed after each request

HTTP 1.1 brought critical performance optimizations and cleared a lot of ambiguities with earlier versions of HTTP. Such as:

- Connection re-use using keepalives

- Chunked encoding transfers

- Byte-range requests

- Additional caching mechanisms

- Transfer encodings

- Request pipelining

HTTP 2 - retains semantics with improved performance

## Chapter 10 - Primer on Web Performance

Check the resource waterfall using [webpagetest](http://www.webpagetest.org)

Different browsers implement different logic for when and in which order individual resource requests are dispatched. As a result, the performance of the application also depends on browser.

3 pillars of performance:

- Computing

- Rendering

- Networking

For fetching hunders of small packets/resources, bandwidth doesn't matter(much), as we are bound by latency.

This is even more true wireless/mobile networks where latency vary much greatly.

### Syntetic and real user performance measurement

Identify key performance metrics and set budgets for each of them. Raise an alarm when budget is exceeded.

Profiling APIs

- User Timing
- Navigation Timing
- Resource Timing

### Browser Optimizations

4 techniques employed by most browsers:

- Resource pre-fetching and prioritization
- DNS pre-resolve
  Likely hostnames are pre-resolved ahead of time to avoid DNS latency on future HTTP requests
- TCP pre-connect
  Following a DNS resolution, browser may speculatevly open a TCP connection in anticipation of an HTTP request
- Page pre-rendering
  Some browsers can let us hint the next destination and can pre-render the page for snappy user navigation

For more, see [google performance networking](http://hpbn.co/chrome-networking)

Extra notes:

- Critical resources such as CSS and JS should be discoverable on document as early as possible
- CSS should be delivered as early as possible to unblock rendering and JS execution
- Non critical javascript should be `deferred` to avoid blocking DOM CSSOM construction
- HTML document is parsed incrementally by the parser; thus, it should be periodically flushed for best performance

We can further hint the browser with:

> link rel="dns-prefetch", "subresource", "prefetch", "prerender

## Chapter 11 - HTTP/1.x

Optimizations and new features:

- Allow connection reuse
- Chunked transfer encoding to enable streaming
- Request pipelining to allow parallel request processing
- Byte serving to allow range-based resource requests
- Improved and much better-specified caching mechanisms
...

To keep in mind:

- Reduce DNS lookups  
  Every hostname resolution requires a network roundtrip, imposing latency
- Make fewer HTTP requests
- Use CDN
- Add an Expires header and configure Etags  
  Relevant resources should be cached to avoid re-requesting on each and every page.

> Look for `head-of-line-blocking` both on TCP as well as on servers

### Using multiple TCP connections

Most browsers allow up to 6 connections per HOST.

Limiting the maximum number of TCP connections, browser provides a safety cgecj against the DoS attack. In absence of such limit, client can saturate all available resources on the server.

### Multiplexing

Multiplexing == two way communication.

HTTP 1.x was one way - you could only communicate in one direction at a time(request/response).

### Domain Sharding

To get around the absence of multiplexing, some developers opt in to serve the assets over multiple hosts, to which they can open up-to ~ 6 TCP connections.

Difficult to use workaround.

Shards are CNAME DNS records pointing to the same IP. Browser connection limitations are enforced with respect to HOSTNAME, not the IP.

### Concatenation and Spriting

- Concatenation
  Multiple JS and CSS files are bundled together into a single resource.

  `Benefits`: Reduced networking overhead, multiple responses over single transfer
  `Drawbacks`: Not all resources may be needed in initial page load, change in one part of the content would invalidate the whole cache.
- Spriting
  Multiple images are combined into a larger, composite image
  `Benefits`: Same as above ^
  `Drawbacks`: Extra CSS to handle all these separation, + above ^

### Things to think about

- Is application blocked on many small, individual resources?
- Can application benefit from selectively combining some requests?
- Will lost cache granularity affect users?
- Will combined image assets cause high memory overhead?
- Will time to first render suffer from delayed execution, and if it's important for the application?

> Consider inlining resources that have sizes between 1-2 KB, as resources below this threshold incur higher HTTP overhead than the resource itself.

Consider Inlining:

- When files are small and limited to specific pages

Consider bundling:

- When small files are often reused between different pages

Keep them separate:

- When small files have high update frequency

Minimize protocol overhead by reducing the size of HTTP cookies

## Chapter 12 - HTTP 2.0

Primary goals were:

- Reduce latency by request multiplexing(bi directional flow)
- Minimize protocol overhead by compressing HTTP headers
- Add support for server prioritization and server push

### Binary Framing

HTTP/2 uses binary framing for HEADERS/DATA in comparison with HTTP/1 which
used just ASCII.

Latter is less performant but easier to inspect. For HTTP/2 inspection, you would need
a tool such as Wireshark.

### Multiplexing

HTTP/1's drawback was that only one response could've been delivered at a time per connection. HTTP/2 eliminates this by allowing multiple requests and responses over a single connection. This removes the need for concatenated files, image sprites, and domain sharding for asset delivery over multiple hosts(to go around the 6 connection limit per HOST).

### Stream Prioritization

Client can hint each request's priority to server by assigning an integer value between 1-256, as well as to
define any dependencies between streams, allowing server to process what is most important first.

### Server Push

With HTTP/2 servers now can send multiple responses/resources to the client, without receiving any requests. For instance, when client reads an HTML response, it will be requesting additional resources such as CSS and JS, but server already knows that, thus pushes those to client ahead of the time.

Client may opt out of these, in case it already cached those resources or it simply doesn't want them. To avoid unnecessary pushes, server first sends PUSH_PROMISE frame, which contains HTTP headers for the resource. If client wants them, it will be followed by the DATA frames, which will contain the resources.

> HTTP PUSH is dead https://evertpot.com/http-2-push-is-dead/

This eliminates the need to `inlining` resources in HTTP/1, a popular optimization technique to avoid waiting for the request from client to response with those resources.

Difference is that in HTTP/1, this was a `forced push`, which client had no way to decline.

### Header Compression

HTTP/2 brought header compression to which in HTTP/1 the plain text headers resulting in 500-800 bytes of overhead per each request, sometimes in kilobytes when cookies were in use.

HTTP/2 uses HPACK compression format.

### Example negotiation


```http
GET /page HTTP/1.1
Host: server.example.com
Connection: Upgrade, HTTP2-Settings
Upgrade: h2c (1)
HTTP2-Settings: (SETTINGS payload) (2)

HTTP/1.1 200 OK (3)
Content-length: 243
Content-type: text/html

(... HTTP/1.1 response ...)

          (or)

HTTP/1.1 101 Switching Protocols (4)
Connection: Upgrade
Upgrade: h2c

(... HTTP/2 response ...)
```

1. Initial HTTP/1.1 request with HTTP/2 upgrade header

2. Base64 URL encoding of HTTP/2 SETTINGS payload

3. Server declines upgrade, returns response via HTTP/1.1

4. Server accepts HTTP/2 upgrade, switches to new framing

## Chapter 13 - Optimizing Application Delivery

As we can't reduce the latency, its crucial we make optimizations at transport and application layers, to eliminate unnecessary roundtrips, requests, and minimize the distance that needs to be travelled for each packet(positioning servers closer to the client).

> Optimize TLS and TCP on servers, try to run the latest versions and make configurations changes according to best practices.

Use APMs, measure and assign tasks depending on business goals, treating performance as a `feature`.

### Evergreen performance best practices

1. Reduce DNS lookups
2. Reuse TCP connections
3. Minimize number of HTTP redirects(best is 0)
4. Reduce rountrip times
   Locate servers closer to client
5. Eliminate unnecessary resources or reduce them
   Caching resources on client
   Compress assets during transfer
   Eliminate unnecessary request bytes(cookies)
   Parallelize request and response processing
   Apply protocol specific-optimizations
6. Careful with cookies(don't use them wherever possible) See `cookie-free` origin
7. Compress js/css(gzip) & images

#### Caching resources on client

- Cache-Control header to specify the cache lifetime(max-age)

- Last-Modified and ETag headers to provide validation mechanisms

> Provide both ^ together

### Optimizing for HTTP/2

HTTP/2 enables more efficient use of network resources with help of request/response multiplexing, header compression, prioritization and more.

HTTP/2, especially with its `one request per origin` requires well tuned TCP/TLS optimization.

Eliminate HTTP/1.x workarounds, such as domain sharding, concatenation and image spriting.

### Test HTTP/2 server quality

Naive implementation of an HTTP/2 server, or proxy that speaks the procotol, but without implementing support for features such as flow control and request prioritization, may result in less efficient transport of data.

For instance, it may result in saturating client's bandwidth by sending large low priority resources while client is blocked from rendering the page until it receives higher priority resources such as HTML, CSS, JS.

However, there may also be a case, where `resource A` has high priority but slow response may block `resource B` which would've been sent much faster(head-of-line-blocking).

Thus, a well implemented server should give priority to high priority resources while also interleave lower priority streams if all higher priority streams are blocked.

## XMLHttpRequest

### History

It really doesn't have anything to do with XML. When it was shipped in Microsoft, it was developed with the help of MSXML team, thus, name was stuck.

Initial version was limited:

- Text based data transfers only
- Restrictred support for handling uploads
- No cross-domain requests

Later, a new draft for `XMLHttpRequest Level 2` addressed these limitations by offering:

- Support for timeouts
- Support for binary and text-based data transfers
- Support for application override of media type and encoding of response
- Support for monitoring progress of evens of each request
- Support for efficient file uploads
- Support for safe cross-origin requests

### CORS (Cross-Origin Resource Sharing)

While XHR API allows client to add custom HTTP headers via `setRequestHeader()`, there are number of protected
headers that are off-limits to application code:

- Accept-Charset, Accept-Encoding, Access-Control-*
- Host, Upgrade, Connection, Referer, Origin
- Cookie, Sec-*, Proxy-* and others...

This guarantees that the application cannot impersonate a fake user-agent, user or origin...

By default requests are set `same origin` due to security. Otherwise, arbitrary scripts can access
data on the server that is hosting the application.

However, there are cases when server would like to offer a resource to a script running in a different origin. This is where `CORS` comes in.

- When request is made, browser is automatically adds the `Origin`(http://example.com) header
- In return, server responds with 

```http
<=Response

HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://example.com
```

- If server would skip the `Access-Control-Allow-Origin`, the request on the client side would fail.

> Never return Access-Control-Allow-Origin: * due to security implications

Is this it? No, CORS makes other precautions such as omitting credentials such as cookies and HTTP authentication. There are other headers for allowing them on server side such as `Access-Control-Allow-Credentials`

Similarly, if client would like to read/write custom HTTP headers, it may issue a preflight request(OPTION) as below:

```http
OPTIONS /resource.js HTTP/1.1
Host: http://thirdparty.com
Origin: http://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: My-Custom-Header

<= Response from server>
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://example.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: My-Custom-Header

<= Actual request starts...>
```

#### Monitoring XHR progress

XMLHttpRequest provides events to monitor the request:

- loadstart - transfer has begun (1)
- progress - transfer is in progress (0+)
- error - transfer has failed (0+)
- abort - transfer has been terminated(0+)
- load - transfer is successful(0+)
- loadend - transfer is finished(1)

Streaming support in XHR is limited, look for better options SSE(Server Sent Events), WebSockets...

Be mindful when going with polling mechanism, evaluate your application requirements and decide if it is a correct solution.

## Server Sent Events (SSE)

Server Sent Events enables efficient server-client streaming of text based event data, e.g., real-time notifications or updates generated on server.

This is achieved by EventSource API built inside browsers, which allows clients to receive notifications from the server as DOM events, in `event stream` data format.

This enables much simpler and efficient communication in the browser vs XHR and its manual handling.

- Low latency delivery via single, long-lived HTTP connection
- Efficient browser message parsing with no unbounded buffers
- Automatic tracking of last seen events and auto reconnect
- Client message notifications as DOM events

Example of EventSource usage:

```javascript
var source = new EventSource("path/to/stream-url")

source.onload = function {}
source.onerror = function {}

source.addEventListener("foo", function {
  ...some-custom-logic-on-foo-event
});

source.onmessage = function(event) {
  log_message(event.id, event.data)

  if (event.id == "CLOSE") {
    source.close();
  }
}
```

SSE provides a memory-efficient implementation of XHR streaming. Unlike raw XHR connection, which buffers the full received response until the connection is dropped, an SSE alternative can discard the processed response without accumulating all of them into memory.

### Event-Stream protocol

SSE event stream is delivered as streaming HTTP response. Client initiates an HTTP request and the server responds with a custom `test/event-stream` content-type, streaming UTF-8 encoded event data.

Example:

```http
=> Request

GET /stream HTTP/1.1
HOST: example.com
Accept: test/event-stream

=> Response
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/event-stream
Transfer-Encoding: chunked

# 15 seconds retry time on dropped requests
retry: 15000

data: Some text based data

data: {"message": "Some Json payload"}

event: foo
data: Some data which is associated with event foo

id: 42
event: bar
data: Some data which has id of 42 and is associated with bar event

id: 43
data: Generic event with id of 43
```

- Event payload is one of group of adjacent data fields
- Responses are separated by an empty line in between
- Events may carry an optional ID and event type string

Event-Source API is designed for a simple text based data transfer. However, as it supports and transferred over HTTP, it's capable of compression. Thus transferring binary data encoded in base64(with 33% byte overhead), is still doable, though, inefficient.

For binary transfers, WebSockets would be much more suitable.

> SSE is a high-performance transport for server-to-client streaming of text based real-time data; messages can be pushed at the moment they are made available at the server. There is minimum message overhead(long-lived connections, event-stream protocol, gzip compression), and the browser handles all the message parsing and auto reconnect.

Two limitations:

- Only from server to client, thus no way to stream requests
- Specifically designed for UTF-8 data. Binary transfers even though possible, is inefficient.

> Be mindful about long lived connections, eliminate unnecessary ones to avoid putting a strain on resources at the client side and of the physical devices(such as battery)

Possible issues may arise from intermediaries such as firewalls, web proxies where the communication can break. Considering delivering an SSE event-stream over a TLS connection.

