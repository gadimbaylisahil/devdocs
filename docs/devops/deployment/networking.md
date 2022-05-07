# Networking

Available ports:

0–1023 TCP/IP Well Known Port Numbers  
1024–49151 Registered/User Ports assigned by the IANA  
49152–65535  Dynamic/Private ports suggested for private use  

## IP

`ip addr`

### Interfaces

* LO: loopback, points to machine itself, localhost.

* We may have other interfaces such as eth0, eth1, wlan, enp5s0, wlp3s0(wlan) and so on.  

Loopback is always same. This way, applications and scripts always knows the address of the localhost.  

Interfaces are followed by their status.

* LOOPBACK: The interface is a loopback. It points back to the machine itself.
* UP: The interface is enabled and up on both sides.
* DOWN: The interface is disabled.
* LOWER_UP: The interface is working on the Link Layer as LOWER simply refers to the “lower layer”. For Ethernet, this would be Ethernet cable being plugged in.
* BROADCAST: The interface can be used for broadcasting. A packet can be sent to all hosts at once.
* MULTICAST: The interface can be used for multicasting. A packet can be sent to multiple hosts at once.
* NOARP: ARP is disabled. As we learned before, this means we cannot find other hosts’ MAC addresses given an IP address using ARP (the protocol).

It is then followed by some extra information:

* mtu:Maximum Transmission Unit refers to the largest block of data it’s possible to send over as one unit on a given medium (larger ones need 72 to be broken down). 1500 bytes is the default for the Ethernet networks for being compatible with older networks. For 1 gigabit and bigger, we might want to see this number increased.
* qdisc: QDisc refers to the Linux abstraction for traffic queues. Each network interface works with a queue of incoming packets and QDisc specifies the model that will be used to queue and distribute the packets efficiently. In our case, it’s fq_codel which is Fair Queuing (FQ) with Controlled Delay (CoDel).
* group: Group specifies a user-defined label that can logically group interfaces. In our case, we don’t have any groups, so it’s default.
* qlen:QLen specifies the length of the receive queue. If the queue is not big enough, some packets might get lost.

### Setting specific interface DOWN

`ip link set eth1 down`

## Hostnames

Hostname and domain names together forms **Fully Qualified Domain Names(FQDM). E.g server1.example.com server2.example.com

```shell
$ host google.com
google.com has address 172.217.23.206
```

Hostnames are saved in `/etc/hosts` file. To resolve a domain name or hostname, this file is checked first. So we better have the addresses mapped to hostnames to avoid resolution times.

It's editable but may be managed by other services/tools. Such as Networking Service Deamon. It may also be managed differently depending on VPS service providers.

`hostnamectl`  

If mapping is not found, `/etc/resolv.conf` file is checked to find a domain name server:

```shell
$ cat /etc/resolv.conf
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
```

## Posts and Sockets

See all services with their respective ports and transfer protocol.

```shell
cat /etc/services
```

### See what ports are occupied and running with SS

```shell
  Utility to investigate sockets.
  More information: https://manned.org/ss.8.

  - Show all TCP/UDP/RAW/UNIX sockets:
    ss -a -t|-u|-w|-x

  - Filter TCP sockets by states, only/exclude:
    ss state/exclude bucket/big/connected/synchronized/...

  - Show all TCP sockets connected to the local HTTPS port (443):
    ss -t src :443

  - Show all TCP sockets listening on the local 8080 port:
    ss -lt src :8080

  - Show all TCP sockets along with processes connected to a remote ssh port:
    ss -pt dst :ssh

  - Show all UDP sockets connected on specific source and destination ports:
    ss -u 'sport == :source_port and dport == :destination_port'

  - Show all TCP IPv4 sockets locally connected on the subnet 192.168.0.0/16:
    ss -4t src 192.168/16
```

### Generate SSL certs for localhost

`openssl req -x509 -sha256 -nodes -newkey rsa:2048 -days 365 -keyout localhost.key -out localhost.crt`