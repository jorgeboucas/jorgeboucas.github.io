---
published: true
title: ssh tunnel
layout: post
---

`target ip` = ip of the machine you want to login into.

`gateway` = middle machine which tunnels the connection.

```
ssh -L 5900:<target ip>:5900 <user>@<gateway>
```
Once the tunnel is set:
```
open vnc://localhost 
```