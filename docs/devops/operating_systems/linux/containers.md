# Containers in Linux

## File systems

Containers have their own filesystems.

Container images are tarball of a filesystem.

Building image:


1. Start with base OS
2. Install dependencies
3. Configure
4. Make a tarball of the whole filesystem

Running image:

1. Download tarball
2. unpack it into a directory
3. Run a program and pretend that the directory is the whole filesystem

## Container in 15 lines of bash

```bash
# Download image
wget bit.ly/fish-container -O fish.tar
mkdir container-root; cd container-root

# Unpack image into directory
tar -xf ../fish.tar

# Generate random cgroup name
cgroup_id="cgroup_$(shuf -i 1000-2000 -n 1)"

# Create cgroup
cgcreate -g "cpu,cpuacct,memory:$cgroup_id"

# Set CPU and memory limits
cgset -r cpu.shares=512 "$cgroup_id"
cgset -r memory.limit_in_bytes=1000000000 "$cgroup_id"

# use the cgroup
# make and use namespaces
# change root directory
# use the right proc
# change hostname
# start fish
cgexec -g "cpu,cpuacct,memory:$cgroup_id" \
  unshare -fmuipn --mount-proc \
  chroot "$PWD" \
  /bin/sh -c "
    /bin/mount -t proc proc /proc &&
    hostname container-fun-times &&
    /usr/bin/fish"
```

## Containers = Processes

Containers is group of Linux processes. In Mac, containers actually run within Linux virtual machine.

Container processes can do anything normal process can, however, they usually have restrictions:

- Differed PID namespace
- cgroup memory limit
- cgroup cpu limit
- different root directory
- not allowed to run `some` system calls
- limited capabilities

## Container kernel features

Allowing containers to have their own namespaces for:

- Network
- PIDS
- Hostname
- mounts
- users
- more

**Cgroups** to limit resource usage for group of processes

**seccomp-bpf** to prevent dangerous system calls

**overlay filesystems** sharing layers between filesystems to reuse disk space

**pivot_root** set a process' root directory to a directory with the contents of the image

## pivot_root

Containers are tarball of the filesystem. We can make use of the tarball:

When we `chroot`, it will open `/usr/bin/redis` within `/fake/root`.

```bash
mkdir redir; cd redis
tar -xzf redis.tar
chroot $PWD /usr/bin/redis
```

Containers use `pivot_root` instead of chroot.

## Container Registeries

Be careful when using public images.

Registeries let you download just the layers you need.

There are public and private registeries.

## cgroups

Cgroups refer to groups of processes and can have limits on physical resources such as CPU and memory.

## namespaces

Containers have different PID namespaces, so `ps aux` will return different results than running on the host OS.

Outside container, means just using the default namespaces.

You can see namespace of the process by `cat /proc/PID/ns`

`lsns -p PID`

Every process has 7 namespaces

## How to make a namespace?

Processes use their parent's namespace by default, but we can switch namespaces anytime.

*Namespace sys calls*:

- clone - make a new process
- unshare - make + use a namespace
- setns - use an existing namespace

Each namespace has its own manpage.

`man network_namespaces`

`clone` lets you create a new namespace for a child process.

Run a command in the same namespace as PID:

```bash
nsenter -t PID --all COMMAND
```

## Container IP addresses

Containers often get their own IP. They use private IP addresses, because they are not on the public internet.

Cloud providers have systems to make container IPs work. In AWS these called `elastic network interface`

## Capabilities

`man capabilities`

By default, containers have limited capabilities. You can get capabilities of a process by `getpcaps PID`

You can use `getcap / setcap` to get and set capabilities.

## seccomp-bpf

All programs use system calls. seccomp-BPF lets you to run a function before a system call. It decides if a syscall is allowed.

Docker blocks dozens of syscalls by default. 2 ways to block dangerous syscalls:

1. limit container's capabilities
2. set a seccomp-bpf whitelist

## configuration options

Several things that you should configure when setting a container:

1. Map a port to the host
2. Mount directories from the host
3. Set capabilities
4. Add seccomp-bpf filters
5. Set memory/cpu limits
6. Use the host network namespace(usually default is to use a new network namespace)

