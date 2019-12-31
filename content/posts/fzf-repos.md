---
title: "Fuzzy-searching GitHub or GitLab repositories"
subtitle: "Using fzf to quickly find and open projects in the browser"
date: 2019-04-11T21:10:51+08:00
tags: ["github", "gitlab", "fzf", "shell"]
---

Here's a script that will open a GitHub repository in your browser:

```sh
#!/bin/sh -

if [ -z "$1" ]; then
	echo "usage: github repo-slug" >&2
	exit 1
fi

open "https://github.com/$1"
```

(`open` is macOS specific, on other OSes you'll use something else, perhaps the
`$BROWSER` variable, `sensible-browser` or `xdg-open` instead.)

It's not particularly interesting on it's own, but what if we could fuzzy search and autocomplete
the repository name?

<script id="asciicast-240294" src="https://asciinema.org/a/240294.js" async></script>

That's possible by using [programmable completion with
`fzf`](https://github.com/junegunn/fzf/wiki/Examples-(completion))!

`fzf` is a really useful utility that I rely on frequently in my day-to-day:

{{< linkpreview title="junegunn/fzf" description="🌸 A command-line fuzzy finder"
url="https://github.com/junegunn/fzf" >}}

Here's my completion script for Bash:

```bash
#!/usr/bin/env bash

cache_dir=${XDG_CACHE_DIR:-$HOME/.cache}
mkdir -p "$cache_dir/fuzzy-repo-finder"
projects_cache="$cache_dir/fuzzy-repo-finder/github_projects"

_github_get_projects() {
	if [ -z "$GITHUB_USERNAME" ]; then
		echo "error: \$GITHUB_USERNAME not set" >&2
		return 1
	fi

	# create empty projects cache
	[ -f $projects_cache ] && cat $projects_cache || touch $projects_cache

	if [ -z "$GITHUB_ACCESS_TOKEN" ]; then
		echo "warning: \$GITHUB_ACCESS_TOKEN not set: only showing cached projects" >&2
		return
	fi

	# fetch the 100 most recently projects we have edit access to
	curl -u "$GITHUB_USERNAME:$GITHUB_ACCESS_TOKEN" --header "Accept: application/vnd.github.v3+json" "https://api.github.com/user/repos?sort=updated&per_page=100" 2> /dev/null |
		jq -r ".[] | .full_name" 2> /dev/null | # extract the repo slug without quotes
		sort |                                  # filter out projects which are already in the cache
		comm -13 $projects_cache - |            # append to the projects cache
		tee -a $projects_cache &&
		sort -o $projects_cache $projects_cache # keep the projects cache sorted  
}

_fzf_complete_github() {
	_fzf_complete "" "$@" < <(_github_get_projects)
}

_fzf_complete_github_notrigger() {
	FZF_COMPLETION_TRIGGER='' _fzf_complete_github
}

[ -n "$BASH" ] && complete -F _fzf_complete_github_notrigger -o default -o bashdefault github
```

The completion is based on your 100 most recently updated GitHub repositories
fetched from the [GitHub
API](https://developer.github.com/v3/repos/#list-your-repositories) and cached
locally.

While working on this, I learnt about the [`comm`
utility](https://en.wikipedia.org/wiki/Comm), which is pretty cool! I used it
to only add new repositories to the list of available completion candidates.

I actually crated this for my work GitLab, and adapted it for GitHub later.
Here's the GitLab version which is the same except for how the available
repositories are fetched from the [GitLab
API](https://docs.gitlab.com/ee/api/projects.html#list-all-projects) instead:

```bash
#!/usr/bin/env bash

cache_dir=${XDG_CACHE_DIR:-$HOME/.cache}
mkdir -p "$cache_dir/fuzzy-repo-finder"
projects_cache="$cache_dir/fuzzy-repo-finder/gitlab_projects"

_gitlab_get_projects() {
	if [ -z "$GITLAB_HOST" ]; then
		echo "error: \$GITLAB_HOST not set" >&2
		return 1
	fi

	# create empty projects cache
	[ -f $projects_cache ] && cat $projects_cache || touch $projects_cache

	if [ -z "$GITLAB_ACCESS_TOKEN" ]; then
		echo "warning: \$GITLAB_ACCESS_TOKEN not set: only showing cached projects" >&2
		return
	fi

	# fetch the 100 most recently projects we have edit access to
	curl --header "Private-Token: $GITLAB_ACCESS_TOKEN" "https://$GITLAB_HOST/api/v4/projects?simple=true&per_page=100&min_access_level=30&order_by=updated_at" 2> /dev/null |
		jq -r '.[] | .path_with_namespace' | # extract the path with namespace without quotes
		sort |                               # filter out projects which are already in the cache
		comm -13 $projects_cache - |         # append to the projects cache
		tee -a $projects_cache &&
		sort -o $projects_cache $projects_cache # keep the projects cache sorted  
}

_fzf_complete_gitlab() {
	_fzf_complete "" "$@" < <(_gitlab_get_projects)
}

_fzf_complete_gitlab_notrigger() {
	FZF_COMPLETION_TRIGGER='' _fzf_complete_gitlab
}

[ -n "$BASH" ] && complete -F _fzf_complete_gitlab_notrigger -o default -o bashdefault gitlab
```

The `min_access_level=30` query parameter shows only projects that you have at
least Developer access to ([GitLab access level
docs](https://docs.gitlab.com/ee/api/members.html)).

For completeness' sake, here's the GitLab version of the script to open a project page in the browser:

```sh
#!/bin/sh -

if [ -z "$1" ]; then
	echo "usage: gitlab project" >&2
	exit 1
fi

if [ -z "$GITLAB_HOST" ]; then
	echo "error: \$GITLAB_HOST not set" >&2
	exit 1
fi

open "https://$GITLAB_HOST/$1"
```

`$GITLAB_HOST` is parameterised because my work GitLab is self-hosted.

I've also created a GitHub repo for the code:

{{< linkpreview title="yi-jiayu/fuzzy-repo-finder" description="Fuzzy auto-completion for GitHub and GitLab repositories powered by fzf"
url="https://github.com/yi-jiayu/fuzzy-repo-finder" >}}
