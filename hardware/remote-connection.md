---
title: WiFi Connection Info
layout: article
breadcrumbs:
 - name: 'Hardware'
   url: '/hardware'
---


<!--
**TODO:** cover connection to ESP in detail, and ST listening swaps between EXT serial and ESP serial dependant on last recieved packet
{:.bitbigger.text-danger}
-->

<br>

See [Marty Setup, Calibration and Troubleshooting Guide](https://robotical.io/learn/article/Marty%20Setup%2C%20Calibration%20%26%20Troubleshooting%20Guide/WiFi%20Setup/) for WiFi setup walkthrough & troubleshooting.
{:.feature}


## Hotspot

The endpoints available in "hotspot mode" are as follows:

- A GET of `/scan` returns a JSON-encoded list of the available networks
- A GET of `wifisave?s=Network&p=Password&name=Name` saves the network config (Network, Password) and the new name (Name) on the ESP.

Managed, Unmanaged, and Websocket connections are all available when operating in hotspot mode. JS-based and UDP-based service discovery are not available.


<br>
