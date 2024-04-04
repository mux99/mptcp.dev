---
layout: home
title: Installation
nav_order: 2
nav_titles: true
titles_max_depth: 2
---
## Enabling on linux systems
run the following command:
```bash
sysctl net.mptcp.enabled=1
```

## Force MPTCP
Apps can be forces to use one of the following methods

- [mptcpize](https://www.mankier.com/8/mptcpize)

    `mptcpize run <app command>`  
    This works by modifying the behavior of the underlying lib-c  
    *note: some admins do not like this technique, that is why it is recommended*
    *to update apps to support MPTCP nativelly*

- [GODEBUG](https://go-review.googlesource.com/c/go/+/507375)

    With GO, the lib-c is not used, so `mptcpize` will not work.
    To force MPTCP the environment variable `GODEBUG=multipathtcp=1` can be used.

- [eBPF](https://git.kernel.org/pub/scm/linux/kernel/git/bpf/bpf-next.git/commit/?id=ddba122428a7)

- [systemtap](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-multipath-tcp_configuring-and-managing-networking#preparing-rhel-to-enable-mptcp-support_getting-started-with-multipath-tcp)

## ss commands
The `ss` command on linux systems has an option to list MPTCP sockets `-M`. The
recommended call is `ss -Menita`

- `M`, MPTCP sockets
- `t`, TCP including subflows created by MPTCP
- `n`, prevent the port number to protocol conversion
- `i`, info on the connection TCP_INFO, MPTCP_INFO, etc.
- `e`, more info
- `a`, display all socket not just "established" ones

[man page](https://www.commandlinux.com/man-page/man8/ss.8.html)


## Making use of multiple subflows
For the apps to be able to use multiple streams the kernel needs to be told what
interfaces can be used to do so.

<details markdown="block">
<summary>Manual configuration</summary>

With multiple addresses defined on several interfaces, you want to be able to tell
your kernel "If I select such source address, please use that specific interface+gateway,
not the default ones". You achieve this by configuring one routing table per outgoing
interface, each routing table being identified by a number. The route selection
process then happens in two phases. First the kernel does a lookup in the policy
table (that you need to configure with ip rules). The policies, in our case, will
be For such source prefix, go to routing table number x. Then the corresponding
routing table is examined to select the gateway based on the destination address.

You need to configure several routing tables in the following manner: Imagine you
have two interfaces eth0 and eth1 with the following properties:

```bash
eth0

  IP-Address: 10.1.1.2
  Subnet-Mask: 255.255.255.0
  Gateway: 10.1.1.1

eth1

  IP-Address: 10.1.2.2
  Subnet-Mask: 255.255.255.0
  Gateway: 10.1.2.1
```

Thus, you need to configure the routing rules so that packets with source-IP 10.1.1.2
will get routed over eth0 and those with 10.1.2.2 will get routed over eth1.

The necessary commands are:

```bash
# This creates two different routing tables, that we use based on the source-address.
ip rule add from 10.1.1.2 table 1
ip rule add from 10.1.2.2 table 2

# Configure the two different routing tables
ip route add 10.1.1.0/24 dev eth0 scope link table 1
ip route add default via 10.1.1.1 dev eth0 table 1

ip route add 10.1.2.0/24 dev eth1 scope link table 2
ip route add default via 10.1.2.1 dev eth1 table 2

# default route for the selection process of normal internet-traffic
ip route add default scope global nexthop via 10.1.1.1 dev eth0
```
With this, your routing table should look like the following:

```bash
mptcp-kernel:~# ip rule show
0:      from all lookup local
32764:  from 10.1.2.2 lookup 2
32765:  from 10.1.1.2 lookup 1
32766:  from all lookup main
32767:  from all lookup default

mptcp-kernel:~# ip route
10.1.1.0/24 dev eth0  proto kernel  scope link  src 10.1.1.2
10.1.2.0/24 dev eth1  proto kernel  scope link  src 10.1.2.2
default via 10.1.1.1 dev eth0

mptcp-kernel:~# ip route show table 1
10.1.1.0/24 dev eth0  scope link
default via 10.1.1.1 dev eth0

mptcp-kernel:~# ip route show table 2
10.1.2.0/24 dev eth1  scope link
default via 10.1.2.1 dev eth1
```
</details>