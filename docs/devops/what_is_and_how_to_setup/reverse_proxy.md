# Reverse Proxy

Sits in front of web server and receives all requests before they reach
the origin server.

Used to enhance:

- Security
- Reliability
- Performance

Difference between reverse and forward proxy is that reverse is used by web server
while forward proxy used by the user/client.

They are very similar but work differently.

Besides directing requests to different web servers, reverse proxy can be useful for:

**Load Balancing**, to distribute load effectively over multiple servers to maintain reliability and performance.

**Global server load balancing**, to distribute load across geographical regions effectively to reduce latency for the requests.

**Enhance security**, to protect against the common attacks with tightened firewall as well as to protect anonymity of the origin server.

**Powerful caching**, you can use reverse proxy to cache both static and dynamic content with tools such as Varnish, Ngnix FAST CGI.

**Superior compression**, response payload size is important to reduce both transmission and download latency. i.e We can use gzip compression with the reverse proxy

**Optimized SSL/TLS encryption/decryption**, SSL/TLS termination is taxing for the origin server, by offloading this task to reverse proxy we free up resources on origin server as well as can server clients based on their geographic region for faster termination.

**A/B testing**, including javascript plugins for A/B testing can be slow and impose considerable latencies. With reverse proxy we can use `split_clients` `sticky_routes` for the redirection of the traffic. See more [here](https://www.nginx.com/blog/performing-a-b-testing-nginx-plus/)

**Monitoring and logging**, monitoring and logging the traffic becomes much easier especially when using multiple origin servers.

## Cons of reverse proxy

- Single point of failure in case proxy is down and clients can't access the web servers directly

- When using HTTPs, reverse proxy should have the SSL/TLS private keys to encrypt/decrypt the request/responses which can be compromised.

- If you are using a third party service for the reverse proxy such as `Cloudflare` you are trusting them with all your web sites data.
