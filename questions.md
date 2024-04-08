---
layout: home
title: Q&A
nav_order: 6
nav_titles: true
titles_max_depth: 2
---

## Are there any security & privacy concerns?
MPTCP aims to maintain the same level of security as traditional TCP, with specific
mechanisms to counter common network attacks. While designed to be secure, vulnerabilities,
as with any protocol, can occur. But the Linux kernel is also known to quickly fix
security issues once they have been identified.

Privacy implications, such as the potential for path correlation by servers, are
important considerations.

## Why & when should it be enabled by default?
Here, servers and clients must be considered separately:
- client, the client is the main beneficiary of MPTCP, but it is only worth it to
enable if the user as configured their systems to make use of its multi-path capability.

- servers, on servers it should be enabled by default to allow it's users to choose
whether or to to use MPTCP. Being backward compatible, an MPTCP socket is able to
communicate with a TCP socket and will devolve itself to TCP.

## Are there unsupported TCP options?
MPTCP support most TCP extensions but not yet all of them (the most obscure ones).
It is also documented that it is not yet compatible with KTLS.

## Are there any performance impact to the use of MPTCP?
MPTCP is engineered to improve network resilience and utilization without adversely
affecting the performance of TCP applications. It adds a manageable overhead that
becomes advantageous when leveraging multiple network paths, potentially increasing
throughput and reliability.

## What are the supported operating systems?
Originally seen as predominantly supported by Linux (from kernel version 5.6), the
adoption of MPTCP extends beyond to various platforms including iOS. It is important
to note that iOS implements its own version of MPTCP.

## MPTCP vs. QUIC
MPTCP enhances TCP's functionality at the transport layer by enabling multipath
capabilities, whereas QUIC, built atop UDP, focuses on reducing latency and improving
connection migration. While both propose multipath functionality, their development
and standardization stages differ.

## What about middleboxes?
MPTCP's interaction with middleboxes, which may not properly handle its extensions,
is meticulously designed to ensure fallback to standard TCP when necessary. This
ensures uninterrupted connectivity amidst the presence of NATs and firewalls.

## How should applications handle missing MPTCP support?
If the system doesn't support MPTCP, a proper error will be returned by the kernel when creating the socket. the kernel version and its config can be different at build-time and at run-time (e.g. some builders are using chroot/containers with up to date software, but an old stable kernel). So we think it is better for the kernel to return an error at run-time if it is not supported than trying to guess what the end-user will have at build-time.

## Is MPTCP support limited to Linux systems?
While MPTCP is well-supported on Linux, enabling advanced networking capabilities on other operating systems, like macOS, iOS, and potentially FreeBSD, presents challenges. The Linux platform offers accessible support through the IPPROTO_MPTCP definition, facilitating integration. On iOS, although MPTCP is available, it requires using the platform-specific SDK or private libraries, which may lack public documentation and official support. The situation on FreeBSD is even more complex, with efforts to implement MPTCP lagging behind and currently non-functional.

Given these challenges, extending MPTCP support beyond Linux requires careful consideration of platform-specific limitations and available documentation. The possibility remains open to incorporate support for other systems in the future, using conditional compilation to integrate platform-specific code as needed. This pragmatic approach allows for focusing on Linux initially while keeping the door open for broader support as MPTCP implementation matures across operating systems.