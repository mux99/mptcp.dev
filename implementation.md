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
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight">
<code><span class="color-main">socket</span>(<span class="color-blue">AF_INET</span>(6), <span class="color-yellow">SOCK_STREAM</span>, <span class="color-green">IPPROTO_MPTCP</span>)</code>
</pre></div></div>

`IPPROTO_MPTCP`{: .color-green} is defined as `262`{: .text-yellow-300}, that is 256 more than the 6 of TCP.

In case MPTCP is not supported by the kernel or otherwise disabled, multiple `errno`{: .text-red-200} are set:
- `ENOPROTOOPT`{: .text-red-200}: Protocol not available, linked to `net.mptcp.enabled sysctl`
- `EPROTONOSUPPORT`{: .text-red-200}: Protocol not supported, MPTCP is not compiled on >= v5.6
- `EINVAL`{: .text-red-200}: Invalid argument, MPTCP is not available on kernels < 5.6

*note: Since a program is not always compiled on the system it run's on.*
*It is recommended to manually define* `IPPROTO_MPTCP` *as follows*

```c
#ifndef IPPROTO_MPTCP
#define IPPROTO_MPTCP 262
#endif
```

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

## Quick examples
### MPTCPize
MPTCP comes with a tool called `mptcpize`, it can be used to start apps with MPTCP.
It works by overwriting the underlying lib C. See hot to use it [here](installation.html#force-mptcp)

### C
```c
#include <sys/socket.h>

#ifndef IPPROTO_MPTCP
#define IPPROTO_MPTCP 262
#endif

int s;
s = socket(AF_INET, SOCK_STREAM, IPPROTO_MPTCP)
if (s == -1) s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
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
MPTCP like TCP comes with a variety of options and infos that can be accessed
with `sockopt`. They are aggregated in three structures:

<details markdown="block">
<summary>struct mptcp_info</summary>

```c
//in the structure, they are grouped by wave of addition, meaning you can get away with only
//verifying the offset of the last element in each group.  
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

	__u8	mptcpi_subflows_total;

	__u8	reserved[3];
	__u32	mptcpi_last_data_sent;
	__u32	mptcpi_last_data_recv;
	__u32	mptcpi_last_ack_recv;
};
```
</details>

<details markdown="block">
<summary>struct mptcp_subflow_info</summary>

```c
struct mptcp_subflow_addrs {
	union {
		__kernel_sa_family_t sa_family;
		struct sockaddr sa_local;
		struct sockaddr_in sin_local;
		struct sockaddr_in6 sin6_local;
		struct __kernel_sockaddr_storage ss_local;
	};
	union {
		struct sockaddr sa_remote;
		struct sockaddr_in sin_remote;
		struct sockaddr_in6 sin6_remote;
		struct __kernel_sockaddr_storage ss_remote;
	};
};

struct mptcp_subflow_info {
	__u32				id;
	struct mptcp_subflow_addrs	addrs;
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

The options supported by mptcp are:
- `MPTCP_INFO`, defined as `1`, is used to interact with `struct mptcp_info` *example 1*
- `MPTCP_TCPINFO`, defined as `2`, is used to interact with `struct mptcp_info`
- `MPTCP_SUBFLOW_ADDRS`, defined as `3`, is used to interact with `struct mptcp_info`
- `MPTCP_FULL_INFO`, defined as `4`, is used to interact with `struct mptcp_full_info` *example 2*

**note: depending on the version of MPTCP used, some options might not be present**
the presence or not of an option can be checked in two ways:
- first, if the variable used to store the data is initialized at zero, missing
values will be zero.
- second, the relative position of an entry in the structure can be compared to
the `optlen` value using the `offsetof` function.

<details markdown="block">
<summary>code example 1</summary>

```c
#include <stdio.h>

struct mptcp_info info = {0};
socklen_t info_len = sizeof(struct mptcp_info);
int fd = 0; //initialize with the file descriptor of an existing socket

if (-1 == getsockopt(fd, SOL_MPTCP, MPTCP_INFO, &info, &info_len)) {
	//handle the error here
}

//info_len has the number of bytes written, so by subtracting it to the byte position
//of the field we want and comparing to zero can can be sure that the value
//has been written to.
else if ((int)offsetof(struct mptcp_info, mptcpi_subflows_total) - (int)info_len < 0){
    printf("%u", info.mptcpi_subflows_total);
}
```
</details>

<details markdown="block">
<summary>code example 2</summary>

```c
#include <stdio.h>

struct mptcp_full_info full_info = {0};

//in this example we only look for two subflows, choose an appropriate value for your needs
//so as to not hardcode the value, a prior call to `getsockopt` with `MPTCP_INFO` can
//give the number of subflows currently open with the `mptcpi_subflows_total`.
full_info.size_arrays_user = 2;
struct mptcp_subflow_info subflow_info[2] = {0};
struct tcp_info tcp_info[2] = {0};

socklen_t full_info_len = sizeof(struct mptcp_full_info);
int fd = 0; //initialize with the file descriptor of an existing socket


full_info.size_sfinfo_user = sizeof(struct struct mptcp_subflow_info);
full_info.size_tcpinfo_user = sizeof(struct tcp_info);

full_info.subflow_info = (unsigned long)&subflow_info[0];
full_info.tcp_info = (unsigned long)&tcp_info[0];

if (-1 == getsockopt(fd, SOL_MPTCP, MPTCP_FULL_INFO, &full_info, &full_info_len)) {
	//handle the error here
}
else{
	for (int i = 0; i < MIN(full_info.size_arrays_user, full_info.num_subflows); i++) {
		printf("subflow %d:\n", i);
		printf("\tid: %u\n", subflow_info[i].id);
		printf("\trtt: %u\n", tcp_info[i].tcpi_rtt);
	}

    printf("subflow count: %u\n", full_info.mptcp_info.mptcpi_subflows_total);
}
```
</details>

<!-- ### The infos in more details
- **the number of subflows**, is available in multiple of the fields

	| field name | structure | description |
	| --- | --- | --- |
	| `mptcpi_subflows` | `MPTCP_INFO` | the number of subflows created after the first one |
	| `mptcpi_subflows_total` | `MPTCP_INFO` | the current number of subflows |
	| `num_subflows` | `MPTCP_FULL_INFO` | the current number of subflows |

	*note: in `mptcpi_subflows` correspond to the number of subflows -1 as long as the initial*
	*subflow is still active*

- **==TODO==** I don't have enough of an understanding at this time to assert what are the useful options and describe them. it would be best for someone else to write this part
note: it should be quite concise and follow the structure set above -->