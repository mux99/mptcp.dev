---
layout: home
title: FAQ
nav_order: 6
nav_titles: true
titles_max_depth: 2
---

## Are there any security & privacy concerns?
MPTCP aims to maintain the same level of security as traditional TCP, with
specific mechanisms to counter common network attacks. Find out more in the
[RFC 8684](https://datatracker.ietf.org/doc/html/rfc8684#name-security-considerations).
To be more secure than TCP, some modifications of the protocol would be needed,
e.g. [MPTCPsec](https://inl.info.ucl.ac.be/system/files/infocom_mptpcsec.pdf).

## Why & when should MPTCP be enabled by default?
<details markdown="block">
<summary>Here, servers and clients must be considered separately: MPTCP can be
enabled by default on servers, to be used only when requested, with a minimal
impact ; while on the client side, it might be interesting to notify them it is
being used by default. </summary>

- Clients are usually the main beneficiaries of MPTCP, but it is mainly worth it
  to enable MPTCP when users have [configured](setup.html#using-multiple-ip-addresses)
  their system to make use of its multipath capability. Still, even when only
  one network interface is available, MPTCP can be useful in mobility use-cases:
  when often switching from one network to another without stopping the
  connections. When servers don't support MPTCP, the connection continues in
  "plain" TCP.

- Servers usually don't directly benefit from MPTCP, because they are not moving,
  and with fast and reliable connections. But their client will, and it will be
  useful for the servers too: letting them switch from one network to another
  without disconnection, not to restart a long operation again ; having faster
  connections, not to hold a transfer for a too long time, etc. We recommend
  enabling MPTCP on servers by default to let users choosing whether to use
  MPTCP. When clients don't request to use MPTCP, server applications will
  receive "plain" TCP sockets from the kernel when connections are `accept`ed,
  making the performance impact minimal.
</details> {: .ctsm}

## Are there any performance impact to use MPTCP?
MPTCP is engineered to improve network resilience and utilization without
adversely affecting the performance of TCP applications. It adds a few bytes in
each TCP packet, causing a manageable overhead (~1%) that becomes advantageous
when leveraging multiple network paths, potentially increasing throughput and
reliability.

It is also important to note that, when clients don't request to use MPTCP,
server applications will receive "plain" TCP sockets from the kernel when
connections are `accept`ed, making the performance impact minimal.

## Are there unsupported TCP socket options?
MPTCP support most TCP extensions. It is possible some most obscure ones are not
supported. If it is the case, please document your use-case in a new
[issue](https://github.com/multipath-tcp/mptcp_net-next/issues/).

For example, MPTCP in the Linux kernel is [currently](https://github.com/multipath-tcp/mptcp_net-next/issues/480)
not compatible with KTLS yet.

## What are the supported operating systems?
<details markdown="block">
<summary>MPTCP is supported in the official Linux kernel from version 5.6. Any
applications can [easily use it](implementation.html). The adoption of MPTCP
extends beyond to various platforms including macOS. But... </summary>

The usage of MPTCP on macOS is somehow restricted:
- It is easy only when applications use their
  [SDK](https://developer.apple.com/documentation/foundation/nsurlsessionconfiguration/improving_network_reliability_using_multipath_tcp)
- If not, it looks like applications need to use private libraries (we are not
  even sure the headers are available) with specific functions to create sockets
  that are apparently not documented, e.g.
  [OpenSSH for macOS](https://github.com/apple-oss-distributions/OpenSSH/blob/main/openssh/sshconnect.c#L487).
  (This might change in the future.)

On FreeBSD, there was an ongoing implementation, but that was years ago, and not
working today according to
[this](http://www-cs-students.stanford.edu/~sjac/freebsd_mptcp_info.html).

There are other implementations, but on specific systems (Citrix load balancer,
userspace, etc.): more details [here](http://blog.multipath-tcp.org/blog/html/2018/12/15/apple_and_multipath_tcp.html).

It is possible to use MPTCP on Windows with
[WSL2](https://perso.uclouvain.be/tom.barbette/mptcp-on-windows-with-wsl2/).
</details> {: .ctsm}

## MPTCP vs. QUIC
MPTCP enhances TCP's functionality at the transport layer by enabling multipath
capabilities, whereas QUIC, built atop UDP, focuses on reducing latency and
improving connection migration. While both propose multipath functionality,
their development and standardization stages differ.

Multipath capability in QUIC is not yet standardized. Here is the
[draft](https://quicwg.org/multipath/draft-ietf-quic-multipath.html).
Applications might have more configuration options to be able to select the
different paths to use, and the technique to use to data over the different
paths.

## What about middleboxes?
MPTCP is meticulously designed to ensure fallback to standard TCP when necessary.
This ensures uninterrupted connectivity amidst the presence of NATs and
firewalls (middleboxes) that might remove "unknown" TCP options like MPTCP.

If you notice that MPTCP is not allowed in your network, and "plain" TCP is used
instead, please report this issue to the people in charge of this network: it is
very likely a mistake that MPTCP is not allowed.

## How should applications handle missing MPTCP support?
Applications supporting MPTCP natively should first try to create an MPTCP
socket, then fallback to "plain" TCP in case of errors. If the Linux kernel does
not support MPTCP, a proper error will be returned when creating the socket. The
kernel version and its config can be different at build-time and at run-time
(e.g. some builders are using chroot/containers with up-to-date softwares, but
an old stable kernel). So it is recommended to do such checks at run-time,
letting the kernel returning an error if it is not supported, than trying to
guess at build-time what the end-user will have at run-time.

## What build time checks are needed/recommended?
Since a common practice nowadays is to compile source code on a different
machine, potentially with an older kernel versions, it is recommended not to
restrict MPTCP utilization at build-time other than checking than the target is
Linux.

Note that it might be needed to manually define `IPPROTO_MPTCP`, because old
libC versions might not have it. See the [Implementation Guide](implementation.html)
for more details about that.

## <code>IPPROTO_MPTCP</code> is not defined
Since a program is not always compiled on the system it run's on, it is normal,
and even recommended to manually define `IPPROTO_MPTCP` if it was not already
the case:
```js
#ifndef IPPROTO_MPTCP
#define IPPROTO_MPTCP 262
#endif
```
