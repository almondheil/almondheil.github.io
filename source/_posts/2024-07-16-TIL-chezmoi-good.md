---
layout: post
category: til
slug: chezmoi good
title: Chezmoi is a Good Dotfiles Manager
---

Previously I used [yadm](https://yadm.io/){:target="_blank"} for my dotfiles, but 
I got frustrated with the way it worked across multiple machines. The templating
just didn't have the capabilities that I wanted it to, and I had an itch to try
something new. I decided to switch to [chezmoi](https://www.chezmoi.io/){:target="_blank"},
which promises things I want from my templating, including:

- user-defined variables (unique to each machine)
- conditionals

Chezmoi also has one feature I was really missing while using yadm, which is
the ability to ignore files on some machines and not others. I want to manage my
VPN config on one machine but not another, so it doesn't seem worth it to have this
configuration appear on all machines when it won't be used.

## My setup

These are the chezmoi features I ended up taking advantage of:
- sensitive file encryption with age (one time decrypt)
- templated files based on user-defined config values

I wanted to list some of the things I did that I think are interesting, because
I didn't see them talked about elsewhere.

My big challenge: Grinnell has MathLAN, which is a bunch of machines with NFS-shared
home dirs but nor much structure going on. They all have hostnames `<something>.cs.grinnell.edu`,
but I couldn't find anything in the templates that would let me check if the hostname *contained*
something.

Because of that, I decided to go with user-defined variables. In addition to setting variables for
my email and whether to use GPG with git (two major differences between the two platforms), I
added a "system" variable that I set either to "laptop" or "mathlan". That lets me do things like
not sync my sensitive files to MathLAN, since it's not a very well-protected system.
