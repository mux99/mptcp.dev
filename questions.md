---
layout: home
title: Q&A
nav_order: 4
nav_titles: true
---

## Security & Privacy

MPTCP aims to maintain the same level of security as traditional TCP, with specific mechanisms to counter common network attacks. While designed to be secure, vulnerabilities, as with any protocol, can occur. Privacy implications, such as the potential for path correlation by servers, are important considerations.
Regarding the security implications of the Linux kernel implementations, it is like any software; there might be issues. But the Linux kernel is also known to quickly fix security issues once they have been identified.

## Why & When to Enable by Default

Enabling MPTCP by default on servers is beneficial for enhancing client connectivity without impacting non-MPTCP traffic performance. This default enablement facilitates broader adoption by eliminating the need for manual configuration by server administrators. Importantly, the fallback to standard TCP in environments not supporting MPTCP is a low-cost operation, ensuring that enabling MPTCP does not significantly alter server operations or performance.

## Performance Impact

MPTCP is engineered to improve network resilience and utilization without adversely affecting the performance of TCP applications. It adds a manageable overhead that becomes advantageous when leveraging multiple network paths, potentially increasing throughput and reliability.

## Supported Operating Systems

Originally seen as predominantly supported by Linux (from kernel version 5.6), the adoption of MPTCP extends beyond to various platforms including iOS. It is important to note that iOS implements its own version of MPTCP.

## MPTCP vs. QUIC

MPTCP enhances TCP's functionality at the transport layer by enabling multipath capabilities, whereas QUIC, built atop UDP, focuses on reducing latency and improving connection migration. While both propose multipath functionality, their development and standardization stages differ.

## Middleboxes

MPTCP's interaction with middleboxes, which may not properly handle its extensions, is meticulously designed to ensure fallback to standard TCP when necessary. This ensures uninterrupted connectivity amidst the presence of NATs and firewalls.

MPTCP represents a significant advancement in TCP technology, catering to modern internet's complex network topologies and connectivity environments. Its ongoing development aims for widespread compatibility and realization of its security and performance benefits.
