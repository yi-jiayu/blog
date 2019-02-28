---
title: "Create data URLs from the command line"
subtitle: "Don't upload your files to random web services"
date: 2019-02-28 20:57:07+08:00
---

Yesterday, I was trying to replace an image on GitHub with another to mockup something by editing
its `src` attribute in the browser inspector, but it didn't work because GitHub has implemented a
[Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) (CSP)[^1] which
only allows images from certain sources:

<pre style="white-space: pre-wrap;"><code>Refused to load the image '<span style="background: black;">           </span>' because it violates the following Content Security Policy directive: "img-src https://*.bitstrips.com 'self' data: github.githubassets.com identicons.github.com collector.githubapp.com github-cloud.s3.amazonaws.com *.githubusercontent.com".</code></pre>

Fortunately for what I was doing, GitHub's CSP still allows [data
URLs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs), so I was able to
get around it by inlining my image as a data URL into the `src` attribute instead of linking to an
image from another host.

Something like this was blocked:

```html
<img src="https://blog.jiayu.co/images/SUTD.png">
```

But using a data URL like this still worked:

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACcAAAAJAQAAAACZfgjeAAAABGdBTUEAALGPC/xhBQAAAAJiS0dEAAHdihOkAAAAB3RJTUUH4gYRBR0dahwXXQAAAD1JREFUCNdjYAgNYGFgqL8aX/+HwSE00IUFyI6N/8LAEBoaIsLwPzw0/AsDY0BoCAvD//ir4X8YGBxEQxgAzdQRP6/iZycAAAAldEVYdGRhdGU6Y3JlYXRlADIwMTgtMDYtMTVUMjM6Mjk6MjUrMDg6MDCObV7UAAAAJXRFWHRkYXRlOm1vZGlmeQAyMDE4LTA2LTE1VDIzOjI5OjI1KzA4OjAw/zDmaAAAAABJRU5ErkJggg==">
```

## Creating data URLs

### The basics

To create the data URL, I initially just searched for "data url generator" and checked out the
first few links.

However, it then occurred to be that this wasn't something that you need to use some random
and potentially shady website to do---it's basically Base64 encoding, and you can do that easily
from the command line:

```console
$ base64 < SUTD.png
iVBORw0KGgoAAAANSUhEUgAAACcAAAAJAQAAAACZfgjeAAAABGdBTUEAALGPC/xhBQAAAAJiS0dEAAHdihOkAAAAB3RJTUUH4gYRBR0dahwXXQAAAD1JREFUCNdjYAgNYGFgqL8aX/+HwSE00IUFyI6N/8LAEBoaIsLwPzw0/AsDY0BoCAvD//ir4X8YGBxEQxgAzdQRP6/iZycAAAAldEVYdGRhdGU6Y3JlYXRlADIwMTgtMDYtMTVUMjM6Mjk6MjUrMDg6MDCObV7UAAAAJXRFWHRkYXRlOm1vZGlmeQAyMDE4LTA2LTE1VDIzOjI5OjI1KzA4OjAw/zDmaAAAAABJRU5ErkJggg==
```

Data URLs come in the format `data:<mediatype>;base64,<data>` where `<mediatype>` is an optional
MIME type which defaults to `text/plain;charset=US-ASCII` and `;base64` is also optional but
specified when the data is Base64-encoded (which is true in our case), so just manually prepend an
appropriate header and you're good to go:

```console
$ echo "data:image/png;base64,$(base64 < SUTD.png)"
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACcAAAAJAQAAAACZfgjeAAAABGdBTUEAALGPC/xhBQAAAAJiS0dEAAHdihOkAAAAB3RJTUUH4gYRBR0dahwXXQAAAD1JREFUCNdjYAgNYGFgqL8aX/+HwSE00IUFyI6N/8LAEBoaIsLwPzw0/AsDY0BoCAvD//ir4X8YGBxEQxgAzdQRP6/iZycAAAAldEVYdGRhdGU6Y3JlYXRlADIwMTgtMDYtMTVUMjM6Mjk6MjUrMDg6MDCObV7UAAAAJXRFWHRkYXRlOm1vZGlmeQAyMDE4LTA2LTE1VDIzOjI5OjI1KzA4OjAw/zDmaAAAAABJRU5ErkJggg==
```

I'm using the `base64` tool on macOS which doesn't output line breaks by default---if you're using
`base64` from GNU coreutils you'll need to pass `-w0` to disable line wrapping.

### A bit more sophistication

The above worked well for the occasional conversion of PNG or JPEG images, but while writing this
post I came across this answer on the Unix & Linux Stack Exchange with a more generic solution which
made use of the `file` command to automatically infer an appropriate MIME type:

{{< linkpreview title="How to generate a Data URI from an image file?" description="There are online tools like duri.me that allow to create a Data URI from an image file. Are there any tools that run locally on Linux to do the same?" url="https://unix.stackexchange.com/a/247846" >}}

Inferring the MIME type of a file:

```console
$ file -bN --mime-type SUTD.png 
image/png
```

The author of that answer provided his own script for creating data URLs (copied verbatim from the
answer):

```console
[0 1026 8:29:38] ~ % cat $(which cssify.sh)
#!/bin/sh
mimetype=$(file -bN --mime-type "$1")
content=$(base64 -w0 < "$1")
echo "url('data:$mimetype;base64,$content')"
```

This also wraps the generated data URL with the [CSS url()
syntax](https://developer.mozilla.org/en-US/docs/Web/CSS/url) and uses the GNU coreutils version of
`base64` which requires the `-w0` flag, either of which can be tweaked to your own liking.

My own version which I modified to work on macOS and added some error handling:

```console
$ cat $(which data-url)
#!/bin/sh
if [ -z "$1" ]; then
	echo "usage: data-url file" >&2
	exit 1
fi
mimetype=$(file -bN --mime-type "$1")
content=$(base64 < "$1")
echo "data:$mimetype;base64,$content"
$ data-url SUTD.png 
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACcAAAAJAQAAAACZfgjeAAAABGdBTUEAALGPC/xhBQAAAAJiS0dEAAHdihOkAAAAB3RJTUUH4gYRBR0dahwXXQAAAD1JREFUCNdjYAgNYGFgqL8aX/+HwSE00IUFyI6N/8LAEBoaIsLwPzw0/AsDY0BoCAvD//ir4X8YGBxEQxgAzdQRP6/iZycAAAAldEVYdGRhdGU6Y3JlYXRlADIwMTgtMDYtMTVUMjM6Mjk6MjUrMDg6MDCObV7UAAAAJXRFWHRkYXRlOm1vZGlmeQAyMDE4LTA2LTE1VDIzOjI5OjI1KzA4OjAw/zDmaAAAAABJRU5ErkJggg==
```

Wonder if I'll ever use it again though 🐣

[^1]: I'm not complaining about this---GitHub has written at length about it [here](https://githubengineering.com/githubs-csp-journey/) and [here](https://githubengineering.com/githubs-post-csp-journey/), and there are very good reasons to have a CSP on your website, such as mitigating cross-site scripting (XSS) attacks. Read an introduction to CSPs [here](https://scotthelme.co.uk/content-security-policy-an-introduction/).
