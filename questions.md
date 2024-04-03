---
layout: home
title: Q&A
nav_order: 6
nav_titles: true
titles_max_depth: 2
---

## Are there any security & privacy concerns?
MPTCP aims to maintain the same level of security as traditional TCP, with specific mechanisms to counter common network attacks. While designed to be secure, vulnerabilities, as with any protocol, can occur. But the Linux kernel is also known to quickly fix security issues once they have been identified.

Privacy implications, such as the potential for path correlation by servers, are important considerations.

## Why & when should it be enable by default?
Here, servers and clients must be considerd separately:
- client, the client is the main beneficiary of MPTCP, but it is only worth it to enable if the user as configured their systems to make use of its multi-path capability.

- servers, on servers it should be enabled by default to allow it's users to choose whether or to to use MPTCP. Being bakward compatible, an MPTCP socket is able to comunicate with a TCP socket and will devolve itself to TCP.

## Are there unsuported TCP options?
MPTCP support most TCP extenions but not yet all of them (the most obscure ones).
It is also documented that it is not yet compatible with KTLS.

## Are there any performance impact to the use of MPTCP?
MPTCP is engineered to improve network resilience and utilization without adversely affecting the performance of TCP applications. It adds a manageable overhead that becomes advantageous when leveraging multiple network paths, potentially increasing throughput and reliability.

## What are the supported operating systems?
Originally seen as predominantly supported by Linux (from kernel version 5.6), the adoption of MPTCP extends beyond to various platforms including iOS. It is important to note that iOS implements its own version of MPTCP.

## MPTCP vs. QUIC
MPTCP enhances TCP's functionality at the transport layer by enabling multipath capabilities, whereas QUIC, built atop UDP, focuses on reducing latency and improving connection migration. While both propose multipath functionality, their development and standardization stages differ.

## What about middleboxes?
MPTCP's interaction with middleboxes, which may not properly handle its extensions, is meticulously designed to ensure fallback to standard TCP when necessary. This ensures uninterrupted connectivity amidst the presence of NATs and firewalls.
