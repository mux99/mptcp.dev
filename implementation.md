---
layout: page
title: Implementation guide
nav_order: 4
nav_titles: true
titles_max_depth: 2
---

## The lib c
Since the Linux kernel v5.6, MPTCP can be used simply by selecting MPTCP in the `socket` command.

like this:
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="color-main">socket</span>(<span class="color-blue">AF_INET</span>(6), <span class="color-yellow">SOCK_STREAM</span>, <span class="color-green">IPPROTO_MPTCP</span>)</code></pre></div></div>

`IPPROTO_MPTCP`{: .color-green} is defined as `262`{: .text-yellow-300}, that is 256 more than the 6 of TCP.

In case MPTCP is not supported by the kernel or otherwise disabled, multiple `errno`{: .text-red-200} are set:
- `ENOPROTOOPT`{: .text-red-200}: Protocol not available, linked to `net.mptcp.enabled sysctl`
- `EPROTONOSUPPORT`{: .text-red-200}: Protocol not supported, MPTCP is not compiled on >= v5.6
- `EINVAL`{: .text-red-200}: Invalid argument, MPTCP is not available on kernels < 5.6

## Are you using MPTCP?
A similar function to the following can be used. [source](https://github.com/multipath-tcp/mptcp_net-next/issues/294)

more details in [#installation](installation.html)
<details markdown="block">
<summary>[click to see function]</summary>

```c
bool socket_is_mptcp(int sockfd)
{
	socklen_t len;
	bool val;

	len = sizeof(val);

	/* We should get EOPNOTSUPP (or ENOPROTOOPT in v6) in case of fallback */
	if (getsockopt(sockfd, SOL_MPTCP, MPTCP_INFO, &val, &len) < 0) {
		if (errno != EOPNOTSUPP && errno != ENOPROTOOPT)
			perror("getsockopt(MPTCP_INFO)");

		return false;
	}

	/* no error: MPTCP is supported */
	return true;
}
```
</details>

## Quick exemples
### MPTCPize
MPTCP comes with a tool called `mptcpize`, it can be used to start apps with MPTCP. It works by overwriting the underlying lib C.

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
MPTCP like TCP comes with a variety of options and infos that can be acessed with `sockopt`. They are agregated in two structures:

<details markdown="block">
<summary>struct mptcp_info</summary>

```c
struct mptcp_info {
	__u8	mptcpi_subflows;
	__u8	mptcpi_add_addr_signal;
	__u8	mptcpi_add_addr_accepted;
	__u8	mptcpi_subflows_max;
	__u8	mptcpi_add_addr_signal_max;
	__u8	mptcpi_add_addr_accepted_max;
	__u32	mptcpi_flags;
	__u32	mptcpi_token;
	__u64	mptcpi_write_seq;
	__u64	mptcpi_snd_una;
	__u64	mptcpi_rcv_nxt;
	__u8	mptcpi_local_addr_used;
	__u8	mptcpi_local_addr_max;
	__u8	mptcpi_csum_enabled;
	__u32	mptcpi_retransmits;
	__u64	mptcpi_bytes_retrans;
	__u64	mptcpi_bytes_sent;
	__u64	mptcpi_bytes_received;
	__u64	mptcpi_bytes_acked;
    __u8    mptcpi_subflows_total;
};
```
</details>

<details markdown="block">
<summary>struct mptcp_full_info</summary>

```c
struct mptcp_full_info {
	__u32		size_tcpinfo_kernel;	/* must be 0, set by kernel */
	__u32		size_tcpinfo_user;
	__u32		size_sfinfo_kernel;	/* must be 0, set by kernel */
	__u32		size_sfinfo_user;
	__u32		num_subflows;		/* must be 0, set by kernel (real subflow count) */
	__u32		size_arrays_user;	/* max subflows that userspace is interested in;
						 * the buffers at subflow_info/tcp_info
						 * are respectively at least:
						 *  size_arrays * size_sfinfo_user
						 *  size_arrays * size_tcpinfo_user
						 * bytes wide
						 */
	__aligned_u64		subflow_info;
	__aligned_u64		tcp_info;
	struct mptcp_info	mptcp_info;
};
```
</details>

**note: depending on the version of MPTCP used, some of the above options might not be present**
the presence or not of an option can be checked in two ways:
- first, if the variable used to store the data is initialized at zero, missing values will be zero.
- second, the relative position of an entry in the structure can be compared to the `optlen` value using the `offsetof` function.

### The infos in more details
- **the number of subflows**, is avaliable in multiple of the fields
	- `mptcpi_subflows`:
	- `mptcpi_subflows_max`: correspond to the number of subflows in exess of the initial one.  
	*note: it does not always correspond the the true number -1, if the inital flow was lost*
	- `mptcpi_subflows_total`:
	- `num_subflows`:

- **==TODO==**