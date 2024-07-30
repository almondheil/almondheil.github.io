---
layout: post
category: til
slug: hold open ssh gateway
title: You can hold open your SSH gateway connection
---

At work this summer, there's an SSH gateway we need to log into that (for some reason)
requires password login and refuses to accept SSH keys. Why? I really don't know. Probably
some silly, goofy security regulation.

We constantly need to log in and out of a couple machines that are only accessible through
this gateway, so it gets frustrating to enter my password each time. Sure, you can avoid
a password entry on the second jump with `ssh-copy-id`, but that doesn't help with the gateway.

To the rescue comes one of my instructors, who has had to deal with this frustration for years
longer and has developed an interesting workaround. The solution is to connect to the gateway and
keep this connection open as a socket on your local machine, and then use that socket to jump
across that gateway without having to reauthenticate.

Here's the SSH config you need to add:

```
# This is a host that connects through the gateway
Host server
  HostName server.example.com
  User me
  ProxyJump gateway

# The gateway. This is where the special settings go!
Host gateway
  # Normal config
  HostName gateway.example.com
  User me
  
  # Open a socket in ~/.ssh/sockets, and send an alive signal every 5 minutes to avoid being killed
  # TODO: run `mkdir -p ~/.ssh/sockets` on your local machine to create this directory
  ServerAliveInterval 300
  ControlMaster auto
  ControlPath ~/.ssh/sockets/ssh_mux_%h_%p_%r
```

To use this, SSH into the gateway in one terminal window or tab or tmux pane or whatever you feel like doing.
Then, don't close it. Open a new terminal, and ssh into your server. It will automatically go to the server
login, which in my case is just an SSH pubkey--no password entry required!

That's all there is to it! I adapted this approach from the work machines to my school's servers,
which also have a frustrating authentication method--a Duo Security login. Yayyy, the solution
works for this as well! It's much more versatile than I expected.
