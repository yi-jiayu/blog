---
title: "Conditional Git configuration"
subtitle: "No more accidentally committing with the wrong email in a new repository"
date: 2019-02-14 23:41:38+08:00
---

## The situation

Have you ever created a new Git repository for work on a personal machine or vice-versa, and
accidentally made your first commit with the global Git email? Something like:

```console
$ git config --global user.email
my@personal.email
$ git init
Initialized empty Git repository in /Users/yijiayu/work/project/.git/
$ echo "do stuff" > stuff.txt
$ git add .
$ git commit -m "Initial commit"
[master (root-commit) 340c447] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 stuff.txt
$ git log -1
commit 4c497e117390eadc3bbc7e840d0dcce58bbcdc2b (HEAD -> master)
Author: Jiayu Yi <my@personal.email>
Date:   Thu Feb 14 09:43:53 2019 +0800

    Initial commit
```

Whoops!

The immediate fix for this would be to set your email at the repository level with `git config
user.email my@work.email"` and then amend the commit with `git commit --amend --author="Jiayu Yi
<my@work.email>" --no-edit`, but perhaps there is a more scalable way?

## A solution

With [conditional includes](https://git-scm.com/docs/git-config#_conditional_includes), you can
override your global configuration on a per-directory basis. For example, you could keep all your
work projects inside your `~/work` directory, and set your work email as the committer email for
projects under this directory. Let's see how to do this.

### Creating a directory-specific Git configuration

We need to create a Git config file to apply for projects in `~/work`. The location of this file
doesn't actually matter, but putting it inside the root of the `~/work` directory makes sense.

We just want to override our email, so we'll create `~/work/.gitconfig` with the following content:

```
[user]
	email = my@work.email
```

### Setting up a conditional include for a directory

Next, we need to add some lines to the end of our global Git config file (usually located at
`~/.gitconfig`) to tell Git to refer to our work-specific Git config when when we're under `~/work`:

```
[includeIf "gitdir:~/work/"]
	path = ~/work/.gitconfig
```

This tells Git to load the Git config file at `~/work/.gitconfig` if the `.git` directory for the
repository you are currently working on is located under `~/work/`. When this happens, the settings
it contains will override the global Git configuration defined higher up in the file---in this case
the `user.email` key defined in `~/work/.gitconfig` will override the one in `~/.gitconfig`.

That's it! Now when you create a new repository inside the `~/work` directory, the committer email
for the repository will default to your work email instead of your personal one, without needing to
manually set it.

### Additional notes

The path passed to the `includeIf` directive is used a glob pattern, so it can
contain wildcard characters. 

It also supports a the `gitdir/i` keyword instead of `gitdir`, which will match the provided pattern
case-insensitively.

Refer to the [`git-config`](https://git-scm.com/docs/git-config#_conditional_includes) documentation
for more details.

## Conclusion

Following up from my [previous post]({{< relref "/posts/git-get" >}}) on organising local Git
repositories based on remote URL, conditional Git includes can be combined with a logical directory
structure to automatically share relevant settings between related repositories, simplifying the
cognitive load of switching between multiple projects.

