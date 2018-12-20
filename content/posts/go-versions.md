---
title: "A shell command to list all Go versions"
subtitle: "Using sed to extract terms from web pages"
date: 2018-12-20T09:55:04+08:00
---

Here's a shell command that will list all available Go versions similar to the output of `rbenv install -l` or `nvm ls-remote`:

```shell
wget -q -O- golang.org/dl | sed -n -E 's/.*toggle(Visible)?" id="(go.*)">/\2/p' | sort | uniq
```

Running it gives the following output:

```console
$ wget -q -O- golang.org/dl | sed -n -E 's/.*toggle(Visible)?" id="(go.*)">/\2/p' | sort | uniq
go1
go1.10
go1.10.1
go1.10.2
go1.10.3
go1.10.4
go1.10.5
go1.10.6
go1.10.7
go1.11
go1.11.1
go1.11.2
go1.11.3
go1.11.4
go1.12beta1
go1.2.2
go1.3
go1.3.1
go1.3.2
go1.3.3
go1.4
go1.4.1
go1.4.2
go1.4.3
go1.5
go1.5.1
go1.5.2
go1.5.3
go1.5.4
go1.6
go1.6.1
go1.6.2
go1.6.3
go1.6.4
go1.7
go1.7.1
go1.7.3
go1.7.4
go1.7.5
go1.7.6
go1.8
go1.8.1
go1.8.2
go1.8.3
go1.8.4
go1.8.5
go1.8.6
go1.8.7
go1.9
go1.9.1
go1.9.2
go1.9.3
go1.9.4
go1.9.5
go1.9.6
go1.9.7
$  
```

It works because the [Downloads](https://golang.org/dl/) page at the Go website has a pretty regular structure---each version has its own `<div>` block which happens to follow a specific pattern, either
```html
<div class="toggleVisible" id="go1.11.4">
```
or
```html
<div class="toggle" id="go1.12beta1">
```

Knowing this, it's easy to extract the version number with a regular expression. I used something the following:

```regexp
class="toggle(Visible)?" id="(go.+)"
``` 

And the adapted it to work with `sed`:

```shell
sed -n -E 's/.*toggle(Visible)?" id="(go.*)">/\2/p'
```

The `.*` at the front matches the entire line, replaces it with the content of the second capturing group (`\2`) and prints it (the `p` at the end). The `-n` flag tells `sed` to only print     the matching lines, while `-E` uses regular expressions rather than basic regular expressions[^1].

`wget` is used to download the content of the page at `https://golang.org/dl/`, but another command like `curl` or a file could have been used as input instead.

`sort` and `uniq` are just there to remove any duplicate entries, since right now `go1.12beta1` appears twice in the HTML.

## Final note

I understand that in general we [shouldn't parse HTML with regex](https://stackoverflow.com/a/1732454), but <code style="background: none; padding: 0;">I̛ gues̛s i̘̘̯̱͜t̯̬̤̬̗̬ ̫̬̼͖͎̣w̸o̺̩͇͚͚͇rk͓͓͓͎̱ͅs̵͔̣͕͕͙ t͉͕̭͈͉̣͝ͅḫ̢̜̗͎́i̸̝̮͉̠̲͔͢s͔̖͔̗̗̯̝̖ ̩̞̫͙t̸̠i͎̕m͉̖͢͜e̵̹̜͓̼͉̞͇.</code>

[^1]: `-E` is a *BSD option, but is also accepted as an [undocumented option](https://www.gnu.org/software/sed/manual/sed.html) on GNU sed.
