# Networking

# dig

makes dns queries

# traceroute & ping & mtr

check hops during a package transfer to a given host

check ping latency

mtr is alternative to traceroute

# nmap

explore a network(which ports are open, what hosts are up?)

# netcat

create TCP or UDP connections from command line

`nc -l PORT` lets you start a server and print everything received to STDOUT

`nc IP PORT` be a client, opens a TCP connection to IP:PORT. Use `-u` to open a UDP connection

## Send files with netcat on the same network

receiver: 

`nc -l 8080 > file`

sender:

`cat file | nc IP 8080`

# socat

Proxy basically any two things. Supported:

- tcp sockets
- unix domain sockets
- pipes
- SSL sockets
- files
- processes
- UPD sockets ...

## Order doesn't matter

`socat THING1 THING2`

`socat TCP-LISTEN:1337 TCP:domain.com:80`

# tcpdump

lets you view network packets send and received

# tshark

Packet analysis tool like Wireshark, CLI version.

# ngrep

Like grep, but for network. 

# openssl

tool for doing SSL stuff.

## Inspect a certificate

`opensll -x509 -in FILE.crt -noout -text`

## CSR

To get an SSL certificate for you website, you need to make a file called CSR(certificate signing request).

CA(Certificate Authorities) ask for this.

*How to make one?*

```bash
openssl req -new -sha256 -key FILE.key -out FILE.csr
```

## List of digest commands

`openssl list -digest-commands`

## Generate md5 checksum of the file

`openssl md5 FILE`

## Other misc tools

- rsync: sync files over ssh or locally
- zenmap: gui for nmap
- httpie: friendlier http client
- aria2c: fancier wget
- nftables: new version of iptables
- tcpflow: capture and assemple TCP streams
- links: brower in terminal
- ab/iperf: benchmarking tools
- pof: identify OS of hosts connecting to you
- lsof: what files are open/ports are used

## SSH

### Install your key on a host over SSH

```bash
# This puts your key in .ssh/authroized-keys in host machine
ssh-copy-id user@host
```

### Run a single command

```bash
# This will run the command given and exit
ssh user@host COMMAND
```

### Port Forwarding or SSH tunneling

```bash
ssh user@host -Nfl 3333:localhost:8888
```

Valuable network resources do not usually allow remote SSH access. 

With SSH tunneling/port forwarding your connection to remote SSH server gets forwarded to a local trusted resource within the network. This way, we can focus on security measures to be taken on the intermediary resource, rather than individual resources.

### .ssh/config

Set aliases, username, ssh keys, connection config etc...

### mosh

SSH alternative client, check it for persistent connections

### ssh-agent

so that your SSH key is remembered in the agent and you don't have to retype it.

## ip

lets you view and change network configuration

### change your mac address(good for time limit wifi connections)

```bash
ip link set wlan0 down
ip link set eth0 address 3c:a9:f4:d1:00:32
ip link set wlan0 up
# here you should restart the network manager
```

## ss

Utility to investigate sockets

```bash
# see all open servers
ss -tunapl
```

By default `ss` will show both `listening` and `non-listening` sockets for all protocols, TCP, UPD, Unix Domain Sockets.

### you can use netstat as alternative

## iptables

iptables lets you create rules to match network packets to accept/drop/modify them.

Used for firewalls and NAT(network address translation)

## tc

It stands for `traffic control`, we can control packets, stopping, dropping and slowing them down.

### Making internet slow

Delay packets by 500ms:

`sudo tc qdisc add dev wlp3s0 root netem delay 500ms`

Revert:

`sudo tc qdisk del dev wlp3s0 root netem`

### Netem

Netem is `Network Emulator` which is part of tc that lets you drop, duplicate, delay, corrupt a packet.

See `man netem`