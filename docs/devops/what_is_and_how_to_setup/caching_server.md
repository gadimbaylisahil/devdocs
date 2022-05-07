# Set up memcached on Ubuntu 20.04

## Installation

```shell
sudo apt install memcached libmemcached-tools
```

## Configuration

Configuration is in `/etc/memcached.conf`. By default it's listening on localhost and unless the application is hosted on a different host, we would not need to change it.

In case of remote access, we need to open up the port using firewall.

## Remote access

Change the configuration in `/etc/memcached.conf`:

From `-l 127.0.01` to an `ip address`.

## Allow access with firewall configuration

```shell
sudo ufw allow from ip_address_here to any port 11211
```