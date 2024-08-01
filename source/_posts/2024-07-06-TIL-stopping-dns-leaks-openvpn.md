---
layout: post
category: til
slug: fix openvpn dns leak
title: Fixing an OpenVPN DNS Leak
---

After reinstalling my machine, I was playing around with my VPN settings and
trying to use openvpn on the command line rather than previous methods.
However, my efforts were partially foiled when I realized I had created a DNS
leak. This took me a long time to figure out and nothing seemed to have the
steps I needed, so I decided to add one more guide to configuring this to the
internet.

## What's a DNS leak, and how did I learn I had one?

A DNS leak is when you accidentally expose your browsing history to your ISP
through DNS requests, even though you're connected to a VPN.

You want to use your VPN's DNS servers when you're connected, but in some cases
your computer still can default to using your ISP's servers. Since you're still
sending DNS requests to those ISP servers, that means you're trusting them with
your browsing data (which was probably not your plan if you're connecting to a
VPN).

I realized I had an IP leak by going to <https://ipleak.net>{:target="_blank"},
which is a site
maintained by my VPN provider that tells you lots of information discoverable
from your public IP.

> It's like one of those "My IP location" websites, but it doesn't advertise to
> you and it shows more information than anything else I've come across. It's a
> good tool!

## My solution to the leak

I ended up solving my DNS leak using the update-systemd-resolved script from
<https://github.com/jonathanio/update-systemd-resolved>{:target="_blank"} (not to be confused with
the openvpn-update-resolv-conf script, which is similar in intent but didn't
work for me).

I also tested out vpnfailsafe, but this turned out to not be what I
wanted--when I disconnect my VPN, that's an expected part of the workflow since
none of my school resources work when connected to a VPN. vpnfailsafe treats
this as a fail state and cuts of connectivity altogether, which isn't what I
wanted.

## Notes for installing update-systemd-resolved, from across two distros

I got the update-systemd-resolved script running properly on both Arch and
Fedora 40 because I've been in a silly distro-hopping mood again, so I have some
helpful notes from each distribution that highlight some subtleties in running
the script

### Installing on Arch

I use Tailscale as well, which seemed to conflict with the OpenVPN when both
were running. I saw an unexpected disconnect, but it gave the same SIGINT log
message that happened when pressing Ctrl+C. 

It turned out, I needed to make a pair of scripts that would toggle both
Tailscale and its network interface whenever any OpenVPN configuration started.

/etc/openvpn/up.sh:

```bash
#!/bin/bash

tailscale down
ifconfig tailscale0 down
```

/etc/openvpn/down.sh:

```bash
#!/bin/bash

ifconfig tailscale0 up
tailscale up --accept-routes
```

As you can see, I take down the tailscale0 network interface to be entirely
sure it won't interfere with the tun0 interface the OpenVPN connection will
use.

Then, I had to make these scripts run when an OpenVPN connection went up or
down. I started off with the example config that the script gave, and changed
it to run two commands in sequence. This goes at the end of every .ovpn file.

```bash
# update systemd-resolved before and after connecting
script-security 2
up '/bin/bash -c "/etc/openvpn/up.sh; /usr/bin/update-systemd-resolved"'
down '/bin/bash -c "/etc/openvpn/down.sh; /usr/bin/update-systemd-resolved"'
# run on restarts, and before the device is closed
up-restart
down-pre
```

---

I had some issues where, even though the VPN DNS servers were showing up, the
ISP ones showed up too. This took a lot of steps to figure out, but it turned
out all I needed to do was add one more line to the end of every .ovpn file.

```bash
# send all DNS queries over the VPN
dhcp-option DOMAIN-ROUTE .
```

### Installing on Fedora

All the notes I made when installing on Arch still apply here, but a few things
were completely different--explained below.

---

When I tried to install on Fedora, I was getting an error where
update-systemd-resolved reported that it could not find the ifconfig or ip
commands, which resulted in it failing and saying that the network interface
tun0 did not exist because it was unable to bring it up without the utils
being found.

To fix this, I just had to look back at the install instructions for
update-systemd-resolved. It turned out I just needed to make sure PATH was
set correctly in each .ovpn file, since Fedora installed everything to very
different locations (the problematic paths were in /usr/sbin).

```bash
setenv PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
