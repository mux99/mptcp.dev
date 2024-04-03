---
layout: home
title: Supported apps
nav_order: 7
---
the apps listed bellow support MPTCP natively, other apps can be forced to use it by following istructions [here](installation.html#force-mptcp)

| Name | PR | default? | how to use |
| --- | --- | --- | --- |
| [Lighttpd 1.4](https://www.lighttpd.net/) | [github](https://github.com/lighttpd/lighttpd1.4/pull/132) | no | `server.feature-flags = ( "server.network-mptcp" => "enable" )` |
