---
layout: home
title: Supported apps
nav_order: 7
---
the apps listed bellow support MPTCP natively, other apps can be forced to use it by following instructions [here](setup.html#force-applications-to-use-mptcp)

## linux apps

| Name | Version | Default? | How to use |
| --- | --- | --- | --- |
| [Lighttpd 1.4](https://www.lighttpd.net/) | [1.4.76+](https://github.com/lighttpd/lighttpd1.4/pull/132) | [no](https://redmine.lighttpd.net/projects/lighttpd/wiki/Server_feature-flagsDetails) | `server.feature-flags = ( "server.network-mptcp" => "enable" )` |
| [Apache Traffic Server](https://trafficserver.apache.org/) | [9.2.4+](https://github.com/apache/trafficserver/pull/10701#event-10945740888) | [no](https://docs.trafficserver.apache.org/en/latest/admin-guide/files/records.yaml.en.html) | `server_ports: <port> mptcp` |
| [Envoy](https://www.envoyproxy.io/) | [1.21.0+](https://github.com/envoyproxy/envoy/pull/18780) | [no](https://www.envoyproxy.io/docs/envoy/v1.21.6/api-v3/config/listener/v3/listener.proto#envoy-v3-api-field-config-listener-v3-listener-enable-mptcp) | In the listening config add `"enable_mptcp": "true"`|
| [QEmu](https://www.qemu.org/) | [6.1+](https://lore.kernel.org/qemu-devel/20210421112834.107651-1-dgilbert@redhat.com/) | [no](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html#qapidoc-48) | `<ip>:<port>,mptcp` |

<!-- | [Syzkaller](https://github.com/google/syzkaller) | [?+](https://github.com/google/syzkaller/pull/1579) | ? | ? |
| [Shadowsocks libev](https://github.com/shadowsocks/shadowsocks-libev) | [?](https://github.com/shadowsocks/shadowsocks-libev/pull/2902) | ? | ? |
| [Shadowsocks Rust](https://github.com/shadowsocks/shadowsocks-rust) | [?+](https://github.com/shadowsocks/shadowsocks-rust/pull/1157) | ? | ? | TODO=incomplete information-->

## macOS apps

| Name | Version | Default? | How to use |
| --- | --- | --- | --- |
| [iperf3-darwin](https://software.es.net/iperf/) | [11.0+](https://developer.apple.com/documentation/foundation/nsurlsessionmultipathservicetype?language=objc) | no | `--apple-multipathsvc` |

<!-- | SSH | [?+]() | no | | TODO=not enough info-->

## tools

| Name | Version |
| --- | --- |
| [TCPDump](https://www.tcpdump.org/) | [4.5.0+](https://github.com/the-tcpdump-group/tcpdump/commit/578dd316f3) |
| [Wireshark](https://www.wireshark.org/) | [4.2.4+](https://github.com/wireshark/wireshark/commit/3bc42dbf8e) |

Please open a new [issue](https://github.com/multipath-tcp/mptcp.dev/issues) or a [pull request](https://github.com/multipath-tcp/mptcp.dev/pulls) if you know other apps supporting MPTCP natively. Meaning they are able to use MPTCP without resolving to using techniques like `mptcpize` described [here](setup.html#force-applications-to-use-mptcp).
