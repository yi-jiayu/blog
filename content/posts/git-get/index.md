---
title: "Namespaced Git clones with git-get"
subtitle: "Keeping your projects organised"
date: 2019-02-12 23:35:10+08:00
tags: ["git"]
---

## tl;dr

`git-get` is a tool for cloning Git repositories into unique, namespaced directories based on the remote URL:

{{< linkpreview title="yi-jiayu/git-get" description="Clone git repositories into namespaced directories like go get" url="https://github.com/yi-jiayu/git-get" >}}

## Background

Last weekend, I was trying to clone a fork of [one of my projects](https://github.com/yi-jiayu/rotom-pokedex-bot) on GitHub to check out a pull request locally, when I ran into a small problem: the name was already taken by my original repository.

Of course, I could have checked out the pull request as a branch instead, or simply just cloned the repository under a different name like `rotom-pokedex-bot-1` and called it a day.

## Project management

However, this made me think about where I kept the various projects on my machine:

- Most of the time, it depended on the IDE I was using for the project: inside `~/IdeaProjects` if I worked on it in Intellij IDEA, `~/PycharmProjects` if I used PyCharm and `~/WebstormProjects` if I used Webstorm.
- However, for Go projects it used to be under `$GOPATH`, until I tried using [Go modules](https://github.com/golang/go/wiki/Modules) and moved them out of `$GOPATH` to `~/Documents`.
- After this, I also started creating new projects directly under `~/Documents`, partially to be more IDE-agnostic.
- My work projects are in `~/Documents/companyname`.

Pretty messy!

(In truth, I wasn't bothered too much because I used [`fzf`](https://github.com/junegunn/fzf) and the [`ALT-C` keybinding](https://github.com/junegunn/fzf#key-bindings-for-command-line) let me jump easily to my various projects with just their names, no matter where they were located.)

Nevertheless, I wondered if there was a better way to organise my projects, and took inspiration from Go import paths and `go get`.

## Go import paths

In Go, import paths for packages outside the standard library start with a domain name. A [recent blog post on the Go blog](https://blog.golang.org/modules2019) explains that the reason for this was decentralisation, allowing Go code to be hosted anywhere instead of depending on a central registry. Including a domain name inside import paths let Go make use of an existing system (DNS) to resolve packages.

Running [`go help importpath`](https://golang.org/pkg/cmd/go/internal/help/#HelpGopath) provides a detailed explanation of how import paths are resolved to package source code, but in practice, before Go modules were introduced, what one used to observe when running a command such as `go get github.com/user/project` was a corresponding directory tree `github.com/user/project` getting created under `$GOPATH/src`.

The enforcement of such "fully-qualified import paths" for Go packages eliminates [disputes over](https://docs.npmjs.com/misc/disputes.html) [package names](https://internals.rust-lang.org/t/crates-io-squatting/8031/3), and conveniently maps packages to unique filesystem paths as well. 

I was also looking for such a mapping from project to canonical filesystem path as a solution to my project management problem, and decided to go with a similar approach based on Git remote URLs, coming up with `git-get`[^1].

## Namespacing project directories by origin URL

Despite Git's decentralised nature, most projects using Git have a canonical repository somewhere, usually on a platform like GitHub or GitLab. This is the case both for my personal projects, which are hosted on GitHub, and my work projects, which are hosted on the company Git server. It is this remote URL which can be used to namespace each project.

### Git URLs

The [man page for `git-clone`](https://git-scm.com/docs/git-clone#_git_urls_a_id_urls_a) describes the format of a Git URL, for example:

```
ssh://[user@]host.xz[:port]/path/to/repo.git/

git://host.xz[:port]/path/to/repo.git/

http[s]://host.xz[:port]/path/to/repo.git/

ftp[s]://host.xz[:port]/path/to/repo.git/

[user@]host.xz:path/to/repo.git/

/path/to/repo.git/

file:///path/to/repo.git/
```

Regardless of protocol, it contains two main parts: a hostname and a path to the repository, which can be combined to form a filesystem path as well.

## `git-get` in action

All `git-get` does is extract the hostname and repository path from a remote URL, then constructs a unique filesystem path relative to a directory specified with the `GITPATH` environment variable and clones it there.

For example, the `git-get` repository itself can be cloned over HTTPS from `https://github.com/yi-jiayu/git-get.git`. The hostname here is `github.com`, while the path to the repository is `yi-jiayu/git-get` after dropping the `.git` suffix. Assuming I have set `GITPATH` to `$HOME/git`, `git-get` will clone itself into `$HOME/git/github.com/yi-jiayu/git-get`:

```console
$ git get https://github.com/yi-jiayu/git-get.git
Cloning into '/Users/yijiayu/git/github.com/yi-jiayu/git-get'...
remote: Enumerating objects: 95, done.
remote: Counting objects: 100% (95/95), done.
remote: Compressing objects: 100% (65/65), done.
remote: Total 95 (delta 42), reused 71 (delta 24), pack-reused 0
Unpacking objects: 100% (95/95), done.
```

`git-get` can be run as `git get` (without the hyphen) because Git automatically makes executables on the system `$PATH` prefixed with `git-` available as subcommands.

### Installation

You can ~~clone~~ _get_ the repository as well to build it from source, download a binary from the [Releases page](https://github.com/yi-jiayu/git-get/releases) and put in on your `$PATH`, or install from Homebrew if you are on MacOS with `brew install yi-jiayu/tap/git-get`.

Check out the project on GitHub:

{{< linkpreview title="yi-jiayu/git-get" description="Clone git repositories into namespaced directories like go get" url="https://github.com/yi-jiayu/git-get" >}}

## Appendix

While working on `git-get`, I also learnt how to automate Go project releases with [GoReleaser](https://goreleaser.com/) and how to create [my own Homebrew tap](https://github.com/yi-jiayu/homebrew-tap).

I'm also planning to write about how you can use [conditional includes](https://git-scm.com/docs/git-config#_conditional_includes) to manage different Git configuration (commit emails in particular) across different repositories more easily.

[^1]: Of course, I know Git is not the only VCS around. However, a majority---if not all---of the projects I interact with are tracked with Git.
