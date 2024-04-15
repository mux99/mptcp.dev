---
layout: home
title: Implementation details
nav_order: 5
nav_titles: false
titles_max_depth: 2
---

The MPTCP protocol is described in the [RFC 8684](https://datatracker.ietf.org/doc/html/rfc8684),
and also in the [MPTCP Doc](https://mptcp-apps.github.io/mptcp-doc/mptcp.html),
but they don't describe the internals of the MPTCP implementation in the Linux
kernel.

A new MPTCP socket type is being used. It is the one exposed to the userspace.
The kernel is in charged of creating subflows sockets: they are TCP sockets
where the behavior is modified using TCP-ULP.

MPTCP listen sockets will create "plain" *accepted* TCP sockets if the
connection request from the client didn't ask for MPTCP.

This page needs to be improved, feel free to
[contribute](https://github.com/multipath-tcp/mptcp.dev/edit/main/datails.md).
