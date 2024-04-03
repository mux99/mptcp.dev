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

## Making use of multiple streams
For the apps to be able to use multiple streams the kernel needs to be told what interfaces can be used to do so.

<details markdown="block">
<summary>Manual configuration</summary>

With multiple addresses defined on several interfaces, you want to be able to tell your kernel "If I select such source address, please use that specific interface+gateway, not the default ones". You achieve this by configuring one routing table per outgoing interface, each routing table being identified by a number. The route selection process then happens in two phases. First the kernel does a lookup in the policy table (that you need to configure with ip rules). The policies, in our case, will be For such source prefix, go to routing table number x. Then the corresponding routing table is examined to select the gateway based on the destination address.

You need to configure several routing tables in the following manner: Imagine you have two interfaces eth0 and eth1 with the following properties:

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

Thus, you need to configure the routing rules so that packets with source-IP 10.1.1.2 will get routed over eth0 and those with 10.1.2.2 will get routed over eth1.

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

<details markdown="block">
<summary>Automatic Configuration</summary>

Doing the above each time by hand is very cumbersome. Some alternative automatic solutions are available:

### Using the configuration scripts (Ubuntu/Debian-based systems)

In /etc/network/if-up.d/ you can place scripts that will be executed each time a new interface comes up. We created two scripts:

{% highlight bash %}
* mptcp_up - Place it inside /etc/network/if-up.d/ and make it executable.
* mptcp_down - Place it inside /etc/network/if-post-down.d/ and make it executable.
{% endhighlight %}

These scripts work most of the time. They use environment variables to configure the routing tables. If they do not work for you, please contact us on the [mptcp-dev](https://listes-2.sipr.ucl.ac.be/sympa/info/mptcp-dev) Mailing-List.

### Automatic configuration with "Multihomed-Routing"

Kristian Evensen <kristian.evensen@gmail.com> developed a set of scripts that integrate well with existing Network Managers to properly configure the multihomed routing. Check it out at [https://github.com/kristrev/multihomed-routing].

### Automatic configuration with ConnMan

Dragos Tatulea <dragos@endocode.com> and Dongsu Park <dongsu@endocode> integrated support for Multipath TCP into ConnMan. Checkout his [github repository](https://github.com/endocode/connman) for access to the source-code

### Another way for Gentoo-based systems

Ondřej Caletka <Ondrej.Caletka@cesnet.cz> created a script for his Gentoo-based system. You can use his script from [github](https://gist.github.com/oskar456/7264828). For any questions, please contact him directly.

### DEPRECATED - Automatic configuration with MULTI

Kristian Evensen <kristrev@simula.no> created a daemon which automatically configures the correct routing tables. You can install his tool from [github](https://github.com/kristrev/multi).
</details>