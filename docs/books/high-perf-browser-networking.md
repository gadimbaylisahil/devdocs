## Chapter 1 - Primer on Latency and Bandwidth

`Latency`

- Time from source sending a packet to the destination receiving it

`Bandwidth`

- Maximum throughput of a logical or physical communication path

`CDN`

- Chief benefits is distributing content around the globe, serving content from nearby location to the client.

`Last-Mile Latency`

- It's often that the last few stops of the packet is the slowest. To connect your home/device/office to internet, your ISP needs to route the cables throughout the neighbourhood, aggregate the signal, and forward it to local routing node. 

Use traceroute to checks the hops to the destination.

```shell
traceroute google.com
```

### Awareness

Speed of light places a hard limit on how fast energy can travel.(300.000 km/s). Current fiber optic lines already can process packets in nearly 200.000 km/s.

So, even reaching the speed of light, would bring just 30-40% improvement only.

Thus, we must improve performance of our applications by architecting and optimizing our protocols and networking code while keeping in mind the limitations of available bandwitdth, and the speed of light.

- Reduce roundtrips
- Move the data closer to client
- Build applications that hides latency through caching, pre-fetching

## Chapter 2 - Building Blocks of TCP

`IP`

- Internet protocol, provides host-to-host routing and addressing

`TCP`

- Transmission Control Protocol, provides reliable network running on unreliable channel. Most used protocol. You are guaranteed that all bytes sent, will be equal to bytes received, in the sent order.

TCP is optimized to accurate delivery rather than speed, thus, it may bring challanges when it comes to optimizing for web performance.

### Three-way handshake

All TCP connections starts with 3 way handshake. Once the handshake is complete, data can flow between client and server. Client can send a packet right after the ACK (last packet of the 3step handshake), and the server `must` wait for ACK before dispatching any data.

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

How bid do flow control(rwnd) and congestion control(cwnd) window values should be?

`Example`

> Assuming minimum of cwnd and rwnd is 16KB and roundtrip time is 100ms

16KB == 131,072 bits

131,072 bits / 0.1s = 1,310,720 bits/s

1,310,720 / 1,000,000 = `1,31` Mbps

Regardless of available bandwidth, TCP connection will not exceed a 1,31Mbps data rate! To achieve a better data rate, we would need to increase the minimum window size or reduce the roundtrip.

### TCP / UPD - When to use what?

TCP brings features to the table such as reliable delivery and in order packet transmission, which is great but comes at the expense of extra latency.

However, not all applications' requirements are same, thus, not all would need reliable delivery and can handle packet losses without sweat.

These, can be video streaming, 3D games, audio streaming, where small losses of packets would not affect the overall experience, however, extra latency would.

This is also why WebRTC uses UDB as its base transport.

### Summary

`TCP`

- 3-way handshake introduces full round trip latency

- Slow-start is applied to each new connection

- Flow and congestion controls regulate the throughput of all connections

- Throughput is regulated by current congestion window size

#### Things to think about

`Increasing TCP's Initial Congestion Window`

Larger starting congestion window allows TCP transfers more data in the first roundtrip and can significantly accelerates the window grwoth - thus, can be critical for short lived connections.

`Slow-start restart`

Disabling slow-start after idle will improve performance of long lived TCP connections.

`Window Scaling`

Enabling window scaling increases the maximum receive window size and allows high-latency connections to acheive better throughput

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