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


