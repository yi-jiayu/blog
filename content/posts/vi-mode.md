---
title: "vi mode"
subtitle: "Supporting vi commands for navigation on my blog"
date: 2018-08-08T23:46:54+08:00
---

Recently, I read an article about the history of the Vim text editor:

{{< linkpreview title="Where Vim Came From" description="Tracing the long lineage of software that brought us Vim." url="https://twobithistory.org/2018/08/05/where-vim-came-from.html" >}}

I found the following excerpt from the article rather amusing:

> There are major websites, including Facebook, that will scroll down when you press the j key and up when you
press the k key—--the unlikely high-water mark of Vim’s spread through digital culture.

This inspired me to add support for navigating my blog with vi commands as well!

The following commands are supported:

- `j`: Move down one line
- `k`: Move up one line
- `}`: Move forward one paragraph
- `{`: Move backward one paragraph
- `G`: Jump to the bottom of the page
- `gg` or `1G`: Jump to the top of the page

The (rather crude) implementation can be found at [/js/vi.js](/js/vi.js). It is highly coupled to my blog's
structure though.

### Related projects

While implementing this functionality on my blog, I looked around to see what else allowed you to use vi-style
navigation on the internet, and came across a few:

- Vimium (Chrome and Firefox extension): [Homepage](https://vimium.github.io/)
[GitHub](https://github.com/philc/vimium)
- vimb (browser): [Homepage](https://fanglingsu.github.io/vimb/) [GitHub](https://github.com/fanglingsu/vimb)
- Vim Vixen (Firefox extension): [GitHub](https://github.com/ueokande/vim-vixen)
- VimFX (Firefox extension): [GitHub](https://github.com/akhodakivskiy/VimFx)
- Saka Key (Chrome and Firefox extension): [Homepage](https://key.saka.io/)
[GitHub](https://github.com/lusakasa/saka-key)
- Tridactyl (Firefox extension): [GitHub](https://github.com/cmcaine/tridactyl)
- Vimperator (Firefox extension): [Homepage](http://vimperator.org/vimperator)
[GitHub](https://github.com/vimperator/vimperator-labs)
- qutebrowser (browser): [Homepage](https://www.qutebrowser.org/) [GitHub](https://github.com/qutebrowser/qutebrowser)

You can find a more comprehensive list over at the [Vim Tips
Wikia](http://vim.wikia.com/wiki/Vim_key_bindings_for_web_browsers).
