---
layout: page
category: project
title: gonote
---

<https://github.com/almondheil/gonote>{:target="_blank"}

---

I wasn't happy with all the parts about how my python script worked, and I wanted to write something in Go
to get a better handle on the language. So this seemed like the perfect candidate!

This is a similar deal as the old [py-note](https://github.com/almondheil/py-note){:target="_blank"},
but with some new features I'm pretty proud of:

- Clearer command line flags, now done with [spf13/cobra](https://github.com/spf13/cobra){:target="_blank"}

- YAML frontmatter, rather than my own hacked-together format. ([adrg/frontmatter](https://github.com/adrg/frontmatter){:target="_blank"} and [yaml.v3](https://pkg.go.dev/gopkg.in/yaml.v3){:target="_blank"})

- Parallel reading of note frontmatters, which helps things work a good deal faster when there's a lot of notes. 
  
  - I got about a 2x speedup over ~10k notes with 4 goroutines, which seemed like a sweet spot to me. I'm probably just bottlenecked by I/O speeds now.
