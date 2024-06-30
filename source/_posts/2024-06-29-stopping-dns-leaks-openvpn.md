---
layout: post
category: til
slug: fix openvpn dns leak
title: One way to fix an OpenVPN DNS leak
---

After reinstalling my machine with Arch, I was playing around with my
installation of [AirVPN](https://airvpn.org/){:target="_blank"} (which is a VPN
I like because it gives you port forwarding, thus the link) and I wanted to see
if I could increase the security of everything more. I decided to connect to
AirVPN through OpenVPN on the command line, and use a GNOME shell extension to
display the VPN status.

Some useful tools I came across were DNS leak checkers, such as the ones at
<https://ipleak.net/>{:target="_blank"} and
<https://dnsleaktest.com/>{:target="_blank"}. These told me that I had a DNS
leak, and this turned out to be a difficult thing to fix for me. All the
information I found was old and a little bit outdated, 

First, let me run down some solutions that I couldn't get to work:

1. OpenVPN has a `block-outside-dns` setting that you can put in the config
   file, but this is only for Windows which is really too bad.

2. I also tried
   [vpnfailsafe](https://github.com/wknapik/vpnfailsafe){:target="_blank"},
   which says it prevents your ISP address from being exposed to the internet
   whether the VPN was on or off.

   This turned out to not be what I wanted for two reasons. First, I don't need
   (or want) my ISP to be hidden when the VPN is disconnected. I'm fine with
   having insecurities when the VPN is disconnected, so this was overkill.

   The second issue is pretty closely-related to the first. For a long time,
   the moment I disconnected from the VPN the `vpnfailsafe` would protect me
   from myself (by disconnecting my WiFi entirely). Eventually I realized there
   was a script to undo its firewall changes, but when I tried to implement
   this I started seeing leaks even when the VPN was connected.

   I don't know what went wrong here, but it ended up being a path I abandoned.

2. The most common script I saw recommended was
   [openvpn-update-resolve-conf](https://github.com/alfredopalhares/openvpn-update-resolv-conf){:target="_blank"},
   but once again this script seemed to leave leaks when I connected to the VPN.

   When I did this, there was never a moment where I stopped DNS leaks at the
   cost of my non-VPN connection. It just never seemed to work.

In retrospect, I think my `/etc/nsswitch.conf` settings (particularly those for
`host`) may have messed the two scripts up. However, I found a different way!

This different way was the
[update-systemd-resolved](https://github.com/jonathanio/update-systemd-resolved){:target="_blank"}
script. From the name, you can see that it's meant to work directly with
`systemd`. It modifies the file `/etc/resolv.conf` just like the other two
scripts do, but it uses DBus updates instead of firewall rules or temporary
files.

This script also has really nice install instructions in the README on github, which I followed pretty closely.

There are also several areas where I deviated from the README, and I want to docment them.
1. I installed from the AUR, which meant some paths were different than listed.
    - The Polkit rules are in
      `/usr/share/polkit-1/rules.d/00-openvpn-systemd-resolved.rules`, not
      `/etc/polkit-1/rules.d/10-update-systemd-resolved.rules`.
    - The script is in `/usr/bin/update-systemd-resolved`, not
      `/usr/local/libexec/openvpn/update-systemd-resolved` or
      `/etc/openvpn/update-systemd-resolved`.

2. I noticed that running `update-systemd-resolved print-polkit-rules ...`
   printed an error stating `print-polkit-rules` was a device, indicating that
   the script interface had changed. I just ignored this issue.

3. I used the second line in the `nsswitch.conf` code block, `hosts: files
   resolve dns myhostname`. I assumed that having multiple `hosts` in the same
   file was not what the script author intended.

4. I didn't have any `myopenvpn-client.service`, which is mentioned in a few areas.
   This was another thing I ignored, and that turned out to be okay.

After I added the necessary lines to each OVPN configuration file, I tested it out!
Everything seemed to run okay, and I could no longer find any DNS leaks on the
sites I used to test for them.
