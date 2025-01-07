---
layout: page
category: project
title: ftag
slug: ftag
---

<https://github.com/almondheil/ftag>{:target="_blank"}

---

A rust script that maintains a database relating files in a directory to tags, and enables searching by these tags.

Meant to be used to tag photos so they can later be queried as a photo library is built.

## 2025-01-06: I found a better one

I recently found out about <https://github.com/ranjeethmahankali/ftag>{:target="_blank"}.
It has the exact same name as my project, but it seems like its development started about a month before I worked on this project.
(I swear I didn't steal the idea, though.)

I think it's really cool that this project exists, and it does a really good job. Here are some features it has that I like better than
my implementation:

- The tags are stored in plaintext
- It has interactive TUI and GUI applications. I haven't tried them yet, but they seem helpful!

It doesn't have a way of adding tags (that's left up to the user, since you can edit the `.ftag` file by hand), but if I end up deciding
I don't want to do that by hand it's a simple format to use and I could write a small extension for myself.

In summary: It feels a little bad that I've made a lesser version of this tool, but I'm looking forward to using this one more!
