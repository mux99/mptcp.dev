---
layout: page
title: Implementation guide
nav_order: 2
nav_titles: true
titles_max_depth: 2
---

## The lib c
Since the Linux kernel v5.6, MPTCP can be used simply by selecting MPTCP in the `socket` command.

like this:
`socket(AF_INET(6), SOCK_STREAM, IPPROTO_MPTCP)`

`IPPROTO_MPTCP` is defined as `262`, that is 256 more than the 6 of TCP.

In case MPTCP is not supported by the kernel or otherwise disabled, multiple `errno`:
- `ENOPROTOOPT`: Protocol not available, linked to `net.mptcp.enabled sysctl`
- `EPROTONOSUPPORT`: Protocol not supported, MPTCP is not compiled on >= v5.6
- `EINVAL`: Invalid argument, MPTCP is not available on kernels < 5.6

## Quick exemples
### MPTCPize
MPTCP comes with a tool called `mptcpize`

### C
```c
#include <sys/socket.h>

#ifndef IPPROTO_MPTCP
#define IPPROTO_MPTCP 262
#endif

int s;

if (-1 == (s = socket(AF_INET, SOCK_STREAM, IPPROTO_MPTCP))) {
    s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)
}
```

### Python
```python
import socket
try:
    IPPROTO_MPTCP = socket.IPPROTO_MPTCP
except AttributeError:
    IPPROTO_MPTCP = 262

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM, IPPROTO_MPTCP)
except:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM, IPPROTO_TCP)
```

## MPTCP infos & options
