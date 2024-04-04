---
layout: home
title: Supported apps
nav_order: 7
---
the apps listed bellow support MPTCP natively, other apps can be forced to use it by following instructions [here](installation.html#force-mptcp)

| Name | Version | Default? | How to use |
| --- | --- | --- | --- |
| [Lighttpd 1.4](https://www.lighttpd.net/) | [1.4.76](https://github.com/lighttpd/lighttpd1.4/pull/132) | [no](https://redmine.lighttpd.net/projects/lighttpd/wiki/Server_feature-flagsDetails) | `server.feature-flags = ( "server.network-mptcp" => "enable" )` |

Please contact us if you know other apps supporting MPTCP natively. Meaning they are able to use MPTCP without resolving to using techniques like `mptcpize` described [here](installation.html#force-mptcp).