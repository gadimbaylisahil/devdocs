# Secure Connections

There are 2 ways of authentication over SSH.

- Password based authentication
- Key based authentication

## OpenSSH

Suite of tools used to have remote access and encrypted communication over SSH. On server side, `sshd` and `ssh-agent` need to be running.

> OpenSSH comes from development of OpenBSD and its developers are still developing it.

## Creating SSH keys

`ssh-keygen -t rsa -a 100 -b 4096 -C "test@example.com"`

Recommended options for keys:

- RSA (prefer larger key size 4096)
- EcDSA(521 bits) - Elliptic Curve
- Ed25519(best) - Elliptic Curve Cryptography

### Change SSH password

`ssh-keygen -f ~/.ssh/id_rsa.new -p`

### Remove SSH password

`ssh-keygen -f ~/.ssh/id_rsa.new -p -N ""`

### ssh-copy-id

Copying over your public key into authorized key list in the server you are connecting to.

`ssh-copy-id -i/home/$USER/.ssh/id_rsa.$NAME.pub root@$IP_ADDRESS`

Using `ssh-copy-id` comes in handy when initially provisioned server comes with password based authentication.

## SSH Config

You may have different SSH keys per different host. We can configure it in ~/.ssh/config.

```shell
cat ~/.ssh/config
  HOST *.fedoraproject.org fedorapeople.org
        IdentityFile ~/.ssh/id_rsa.fedora 
  HOST *.redhat.com rhcloud.com
        IdentityFile ~/.ssh/id_rsa.redhat
```

You can use `ForwardAgent: yes` option to not put your keys in the servers. SSH Agent will handle to keep credentials in memory during connection and we can utilize them at later stages.

## Secure file transfers

### SCP

`scp -P $PORT file.txt user@$SERVER:/destination`  

`scp -r user@example.com:/path/to/dir /home/$USER/Documents/`  

- Straightforward and easy to use
- Have had some security issues which mostly is solved

### Rsync

Also uses SSH underneath.  

`rsync ~/source/* root@192.168.56.100:~/destination`  

Both of these options work with ~/.ssh/config.

## SSH on servers

SSH work both with password based and key based authentication options.

If password based feature is not used we should turn it off in `/etc/ssh/sshd_config`.

Actions to take:  

- Set PasswordAuthentication to `no`

### Authorized Keys

Saved public keys is stored in users' ~/.ssh/authorized_keys file. Then connections made outside will be verified using private identity key.  

Location of this file can be adjusted with `AuthorizedKeysFile` option which is configurable at `/etc/ssh/sshd_config`.  

It would be good idea to change the location of this file to `root`'s .ssh/authorized_keys/$USER in case we want to restrict other user's ability to freely add other keys to access their accounts.  

```shell
# in /etc/ssh/sshd_config
  AuthorizedKeysFile  /etc/ssh/authorized_keys/%u
```

### Known Hosts

Host keys work like authorized keys but for users to connect to verified servers.

On first connection to any particular server OpenSSH saves the public key of the server in ~/.ssh/known_hosts file and any subsequent connections are verified.

If the private key of the server and known public key does not match, we get a warning from SSH.

### SSH Tunneling

SSH Tunneling is simple a way to forwards application ports from clients to servers or vice versa.

Example:

Local database running on port 5432 and is not exposed to internet. We still need to connect to it to maintain and this would be the case where SSH tunneling comes into play.

To locally forward this port over to server, use:

`ssh -L 127.0.0.1:5432:127.0.0.1:5432 $SERVER`.

Local forwarding can be disabled using following settings in ssh_conf.

```shell
AllowTcpForwarding yes|no|local|remote
AllowStreamLocalForwarding no
```
