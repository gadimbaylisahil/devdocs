# Chapter 1 - Primer on Latency and Bandwidth

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

## Awareness

Speed of light places a hard limit on how fast energy can travel.(300.000 km/s). Current fiber optic lines already can process packets in nearly 200.000 km/s.

So, even reaching the speed of light, would bring just 30-40% improvement only.

Thus, we must improve performance of our applications by architecting and optimizing our protocols and networking code while keeping in mind the limitations of available bandwitdth, and the speed of light.

- Reduce roundtrips
- Move the data closer to client
- Build applications that hides latency through caching, pre-fetching

