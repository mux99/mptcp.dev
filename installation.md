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

    {: .note}
    Some admins do not like this technique, that is why it is recommended to update
    apps to support MPTCP natively.

- [GODEBUG](https://go-review.googlesource.com/c/go/+/507375)

    With GO, the lib-c is not used, so `mptcpize` will not work.
    To force MPTCP the environment variable `GODEBUG=multipathtcp=1` can be used.

- [eBPF](https://git.kernel.org/pub/scm/linux/kernel/git/bpf/bpf-next.git/commit/?id=ddba122428a7)

- [systemtap](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-multipath-tcp_configuring-and-managing-networking#preparing-rhel-to-enable-mptcp-support_getting-started-with-multipath-tcp)

## ss commands
The `ss` command on linux systems has an option to list MPTCP sockets `-M`. The
recommended call is `ss -Mani`. To see subflows (if executed as root) `ss -tani`
can be used, subflows are marked as `tcp-ulp-mptcp`.

- `M`, MPTCP sockets
- `t`, TCP including subflows created by MPTCP
- `n`, prevent the port number to protocol conversion
- `i`, info on the connection TCP_INFO, MPTCP_INFO, etc.
- `e`, more info
- `a`, display all socket not just "established" ones

[man page](https://www.commandlinux.com/man-page/man8/ss.8.html)


## Making Use of Multiple Streams
For the apps to be able to use multiple streams the kernel needs to be told what
interfaces can be used to do so. Two steps are required to achieve this:

- **Setting MPTCP Endpoints**
  This operation tells MPTCP which interfaces on the system can be used as path by
  mptcp. To set MPTCP endpoints, you use the `ip mptcp` command. There is basically
  two cases:

  Servers:  
  this is the option to use when you want distant hosts to create new subflow on that interface
  ```sh
  ip mptcp endpoint add <ip address> signal
  ```

  Client:  
  this, on the other hand, tells MPTCP to use this interface to create new subflows.
  ```sh
  ip mptcp endpoint add <ip address> subflow
  ```
  [man page](https://man7.org/linux/man-pages/man8/ip-mptcp.8.html)

- **Configuring Routing**
  When MPTCP knows what interfaces it can use and how to use them, the system needs
  to be told how to route packets for those interfaces. To achieve this, we will
  create a new routing table per interface. We will use `ip route` and `ip rule`.

  For each interface that will use MPTCP, run the following command (change the table number each time)
  ```sh
  ip rule add from <ip address> table <table number>
  ```

  The we configure each table like so:
  ```sh
  ip route add <network ip>/<network mask> dev <interface name> scope link table <table number>
  ip route add default via <default gateway> dev <interface name> table <table number>
  ```

  The last thing to do is to configure the default route for normal internet traffic:
  ```sh
  ip route add default scope global nexthop via <default gateway> dev <exit interface name>
  ```
  [man page](https://man7.org/linux/man-pages/man8/ip-route.8.html)