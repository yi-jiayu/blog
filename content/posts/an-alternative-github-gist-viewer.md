---
title: "An alternative Github Gist viewer"
subtitle: "In case your corporate proxy blocks gists"
date: 2018-06-09T14:23:46+08:00
tags: ["GitHub", "Gist", "git", "HTML", "JavaScript"]
---

[GitHub Gists](https://gist.github.com/) are extremely handy for sharing self-contained chunks of information like code snippets, short scripts and even complete articles.

However, I recently found myself working behind a corporate proxy which blocked access to gists, probably because of data loss prevention concerns. As a result, when searching online for answers I would often come across promising looking gists in my search results, but would also be frustratingly unable to view their contents.

## The solution
After some investigation, I noticed that while `gist.github.com` was blocked, other GitHub domains such as `gist.githubusercontent.com` or `api.github.com` were not. This was promising, because the former is where the raw content for gists is hosted, and the latter is the base URL for the [GitHub REST API](https://developer.github.com/v3/) through which [gists can be accessed as well](https://developer.github.com/v3/gists/).

I ended up building a simple static page which takes a gist ID or URL and grabs its contents from the GitHub API to display:

{{< linkpreview title="Essence - Gist Viewer" description="An alternative GitHub gist viewer" url="https://yi-jiayu.github.io/essence/" >}}
{{< figure src="/images/essence-gist-viewer.png" alt="Screenshot of https://yi-jiayu.github.io/essence/" link="/images/essence-gist-viewer.png" caption="Screenshot" >}}

This allows you to view gists even if `gist.github.com` is blocked, and, since it's read only, hopefully doesn't run afoul of DLP policies (just in case, I also made sure not to mention "proxy" on the page).

### Additional features
- **Shareable URLs.** By using the URL's fragment identifier---the part starting with a `#` symbol---Essence URLs can be shared as-is and will directly load the same gist, for example: [89d5530b693dcde89fdfa9d4dd421863](https://yi-jiayu.github.io/essence/#89d5530b693dcde89fdfa9d4dd421863).
- **Markdown formatting**. Using [Marked](https://github.com/markedjs/marked) and GitHub's own [Primer Markdown](https://github.com/primer/primer/tree/master/modules/primer-markdown) styles, Markdown gists are formatted and look just like on GitHub: [838cb4c84ff6dc45620baf26bad0e231](https://yi-jiayu.github.io/essence/#838cb4c84ff6dc45620baf26bad0e231).
- **Bookmarklet.** Using a bookmarklet which automatically extracts the gist ID from the current page and redirects to Essence, you can open the blocked gist URL first and then view it with one click.

### Possible improvements
- **Display more gist information.** Right now some fields such as author and date published are noticably missing, and it might be nice to see other details such as number of stars and view comments.
- **Handle large gists.** The GitHub API only includes up to 1 MB of individual file contents inline and only list up to 300 files in a gist (although how often do you need to open such large gists anyway?).

### Source

Essence is on GitHub at https://github.com/yi-jiayu/essence.

## Things I learnt along the way
### Anatomy of a raw gist URL
The raw contents of a file in a gist can be found under `https://gist.githubusercontent.com` at (`:filename` can be omitted for the first or only file in a gist):

- `/:username/:gist_id/raw/:commit_hash/:filename`
- `/:username/:gist_id/raw/:blob/:filename`
- `/:username/:gist_id/raw/:filename` (points to the latest revision)

For example,
```
https://gist.githubusercontent.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863/raw/265fe21e0930c18dace1f8da4941fbbe47ba4905/benford.ipynb
https://gist.githubusercontent.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863/raw/265fe21e0930c18dace1f8da4941fbbe47ba4905/
https://gist.githubusercontent.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863/raw/c16ad4ac4597a57876ded6e1288e89efd8e4f532/benford.ipynb
https://gist.githubusercontent.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863/raw/c16ad4ac4597a57876ded6e1288e89efd8e4f532/
https://gist.githubusercontent.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863/raw/benford.ipynb
https://gist.githubusercontent.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863/raw
```
all lead to the same file.

### Looking up Git objects by hash

While looking into the significance of the various parts of the raw gist URLs, I learnt about the [`git-cat-file`](https://git-scm.com/docs/git-cat-file) command:

```
jiayu@lileep ~> set tmpdir (mktemp -d)
jiayu@lileep ~> git clone https://gist.github.com/89d5530b693dcde89fdfa9d4dd421863.git $tmpdir
Cloning into '/tmp/tmp.5O8PrhMFSz'...
remote: Counting objects: 27, done.
remote: Total 27 (delta 0), reused 0 (delta 0), pack-reused 27
Unpacking objects: 100% (27/27), done.
jiayu@lileep ~> cd $tmpdir
jiayu@lileep /t/tmp.5O8PrhMFSz> git cat-file -t 265fe21e0930c18dace1f8da4941fbbe47ba4905
commit
jiayu@lileep /t/tmp.5O8PrhMFSz> git cat-file -p 265fe21e0930c18dace1f8da4941fbbe47ba4905
tree cc3dd4db2bada8494b9718b0d76ac09624074bc8
parent c6b728a0f7e08c8677d133e62fba3cfe35d71b1c
author Jiayu Yi <yi-jiayu@users.noreply.github.com> 1526828087 +0800
committer GitHub <noreply@github.com> 1526828087 +0800

jiayu@lileep /t/tmp.5O8PrhMFSz> git cat-file -t c16ad4ac4597a57876ded6e1288e89efd8e4f532
blob
jiayu@lileep /t/tmp.5O8PrhMFSz> git cat-file -p c16ad4ac4597a57876ded6e1288e89efd8e4f532
{
 "cells": [
  {
...
 "nbformat": 4,
 "nbformat_minor": 2
}
jiayu@lileep /t/tmp.5O8PrhMFSz>
```

### HTML5 form data validation

I got stuck on this for a while wondering why my regular expression to match a GitHub gist ID:
```
PS jiayu@lileep ~> node
> /(?:^|\/)([a-f0-9]+)$/.exec('https://gist.github.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863')
[ '/89d5530b693dcde89fdfa9d4dd421863',
  '89d5530b693dcde89fdfa9d4dd421863',
  index: 32,
  input: 'https://gist.github.com/yi-jiayu/89d5530b693dcde89fdfa9d4dd421863' ]
> /(?:^|\/)([a-f0-9]+)$/.exec('89d5530b693dcde89fdfa9d4dd421863')
[ '89d5530b693dcde89fdfa9d4dd421863',
  '89d5530b693dcde89fdfa9d4dd421863',
  index: 0,
  input: '89d5530b693dcde89fdfa9d4dd421863' ]
> /(?:^|\/)([a-f0-9]+)$/.exec('https://blog.jiayu.co/2018/06/an-alternative-github-gist-viewer/')
null
> /(?:^|\/)([a-f0-9]+)$/.exec('not-a-gist-id')
null
PS jiayu@lileep ~>
```
wasn't working in my `<input>` element's [`pattern`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attr-pattern) attribute:
```html
<input type="text" class="form-control form-control-lg" id="gist_id_or_url_input"
   placeholder="https://gist.github.com/0123456789abcdef" required pattern="(?:^|/)([a-f0-9]+)$"
   title="A gist ID, or the last segment of a gist URL, should be a lowercase hexadecimal string.">
```
{{< figure src="/images/form-validation.png" alt="Unexpected invalid form input" link="/images/form-validation.png" caption="Built-in form validation in Firefox 61.0b12 (64-bit)" >}}

I later found out that it was because the regexp in the `pattern` attribute must match the entire input value for it to be considered valid. From the current version of the HTML5 specification ([HTML 5.2](https://www.w3.org/TR/2017/REC-html52-20171214/)):

> This implies that the regular expression language used for this attribute is the same as that used in JavaScript, except that the `pattern` attribute is matched against the entire value, not just any subset (somewhat as if it implied a `^(?:` at the start of the pattern and a `)$` at the end).
> {{< cite text="4.10.5.3.6. The pattern attribute" url="https://www.w3.org/TR/2017/REC-html52-20171214/sec-forms.html#element-attrdef-input-pattern" >}}

To fix this, I rewrote my regular expression from `(?:^|/)([a-f0-9]+)$` to `(?:.*/)?([a-f0-9]+)`, consuming the leading non-ID part of a gist URL and leaving out the implied `^` and `$` anchors.

### The form `submit` event

Due to my lack of experience with HTML, forms and JavaScript, I initially handled the form submission with a `click` event handler on the submit button. It was only after I wondered why the form validation didn't work that I realised I should listen to the `submit` event on the form element itself instead.

## Appendix

The site on GitHub Pages:

{{< linkpreview title="Essence - Gist Viewer" description="An alternative GitHub gist viewer" url="https://yi-jiayu.github.io/essence/" >}}

GitHub repository:

{{< linkpreview title="yi-jiayu/essence" description="View GitHub gists behind a proxy" url="https://github.com/yi-jiayu/essence" >}}

