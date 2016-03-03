---
Categories:
- coreos
Description: ""
Tags:
- docker
- java
- alpine
date: 2016-03-03T21:00:00+00:00
menu: main
title: java.net.UnknownHostException with Alpine Linux
---

There is no Name Service Switch file in Alpine linux, and java need one for java.net.InetAddress.getLocalHost for example.

```
echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' > /etc/nsswitch.conf
```

Or in a Dockerfile :

```
RUN echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' > /etc/nsswitch.conf
```
