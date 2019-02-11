---
title: "Hot Corners profiles"
subtitle: "Quickly chanaging Hot Corners settings on macOS"
slug: "quickly-configuring-hot-corners-on-macos"
date: 2018-12-24T23:12:29+08:00
image: "preferences.png"
tags: ["awk", "scripting", "automation", "macOS", "pair programming"]
---

My company is big on agile, so we're often encouraged to [work in pairs](https://en.wikipedia.org/wiki/Pair_programming).

One challenge with pair programming on personal machines is that not everyone might be equally comfortable with the configuration. Ignoring the fact that as a [Dvorak keyboard layout](https://en.wikipedia.org/wiki/Dvorak_Simplified_Keyboard) user I'm immediately awkward on a teammate's machine, a setting I use on my own machine that my pairs are often not used to is the Hot Corners.

{{< figure src="preferences.png" caption="Hot corners configuration in System Preferences" class="nooutline" >}}

Hot Corners is a feature on macOS which triggers certain actions when the cursor is moved to the corners of the screen. In the screenshot above, I have my bottom right corner mapped to showing the desktop, so moving the cursor there immediately hides all open application windows---a jarring interruption when one is not expecting it.

I'll usually end up disabling Hot Corners temporarily if it starts to become a problem, but this entailed navigating through a drop down menu four times, once for each corner, and then again when I wanted to re-enable them. Rather than resolving to not rely on them or using a modifier key, I wondered if there was a way to quickly switch between different Hot Corners "profiles". Eventually, I cobbled together a script to do it using the built-in `defaults` command in macOS.

## Setting user preferences from the command line

```
DEFAULTS(1)               BSD General Commands Manual              DEFAULTS(1)

NAME
     defaults -- access the Mac OS X user defaults system
```

On macOS, the `defaults` command allows you to configure various user preferences, often with more control than is available through the user interface. For example, you can use it to set a faster key repeat rate and shorter delay until repeat than is possible through "System Preferences > Keyboard", as detailed in [this StackExchange answer](https://apple.stackexchange.com/a/83923):

```shell
defaults write -g InitialKeyRepeat -int 10 # normal minimum is 15 (225 ms)
defaults write -g KeyRepeat -int 1 # normal minimum is 2 (30 ms)
```

You can find many hidden settings that can be tweaked with `defaults` from a quick web search.

### Getting and saving the current Hot Corners configuration

Using the `defaults read` subcommand, we can view the current Hot Corners configuration (grepping for `vwous` filters out the other irrelevant keys):

```console
$ defaults read com.apple.dock | grep wvous  
    "wvous-bl-corner" = 2;
    "wvous-bl-modifier" = 0;
    "wvous-br-corner" = 4;
    "wvous-br-modifier" = 0;
    "wvous-tl-corner" = 3;
    "wvous-tl-modifier" = 0;
    "wvous-tr-corner" = 12;
    "wvous-tr-modifier" = 0;
```

- `bl`, `br`, `tl` and `tr` refer to the bottom-left, bottom-right, top-left and top-right corners respectively.
- The values for the `wvous-*-corner` keys represent the actions associated with each corner, which can be found either through experimentation or from the comments [here](https://github.com/mathiasbynens/dotfiles/blob/master/.macos#L415).
- The values for the `vwous-*-modifier` keys represent combinations of modifier keys which need to be pressed for the hot corner to trigger as a bit mask: no modifier key is 0, Shift is 2^17 or 131072, Control is 2^18 or 262144, Option is 2^19 or 524288 and Command is 2^20 or 1048576.

I used `sed` to convert the current configuration into a more workable format and saved it to a file:

```console
$ defaults read com.apple.dock | grep wvous | sed -n -E 's/    "(.+)" = (.+);/\1=\2/p' > profile1
$ cat profile1 
wvous-bl-corner=2
wvous-bl-modifier=0
wvous-br-corner=4
wvous-br-modifier=0
wvous-tl-corner=3
wvous-tl-modifier=0
wvous-tr-corner=12
wvous-tr-modifier=0
```

### Applying a saved Hot Corners configuration

We can manually set a hot corner by using the `defaults write` subcommand:

```console
$ defaults write com.apple.dock wvous-bl-corner -int 2
$ defaults write com.apple.dock wvous-bl-modifier -int 0
$ killall Dock
```

This sets the action for the bottom-left corner to open Mission Control without needing a modifier key. After updating the configuration, we need to restart our Dock for it to take effect.

To set all the corners from our saved configuration, we can use `awk` to parse it and invoke `defaults write` with the appropriate arguments:

```console
$ awk -F\= '{
    cmd=sprintf("defaults write com.apple.dock %s -int %s", $1, $2); 
    print cmd; 
    system(cmd);
    }' profile1
defaults write com.apple.dock wvous-bl-corner -int 2
defaults write com.apple.dock wvous-bl-modifier -int 0
defaults write com.apple.dock wvous-br-corner -int 4
defaults write com.apple.dock wvous-br-modifier -int 0
defaults write com.apple.dock wvous-tl-corner -int 3
defaults write com.apple.dock wvous-tl-modifier -int 0
defaults write com.apple.dock wvous-tr-corner -int 12
defaults write com.apple.dock wvous-tr-modifier -int 0
```

- Passing the flag `-F\=` tells awk to use `=` as the field separator.
- We use the `printf` function within awk to prepare our command and store it to a variable, so that we can print it out for feedback and then use the `system` function to run it as an external command.

## Wrapping up

### The finished script

My rudimentary script takes a single argument to the name of a file containing a saved Hot Corners configuration, and looks for a file with that name in a given directory, using it to reconfigure the Hot Corners if found.

```bash
#!/usr/bin/env bash

set -e

config_dir=${HOTCORNERS_CONFIG_DIR:-$HOME/.hotcorners}

if [ -z "$1" ]; then
	echo "usage: hotcorners profile" 1>&2
	exit 1
fi

profile=$1
config_file_path=$config_dir/$profile
if [ ! -f "$config_file_path" ]; then
	echo "couldn't find config file for profile $profile ($config_file_path)" 1>&2
	exit 1
fi

awk -F\= '/^[^#]/ {cmd=sprintf("defaults write com.apple.dock %s -int %s", $1, $2); print cmd; system(cmd)}' $config_file_path
killall Dock
```

An additional regex passed to the awk command also allows for comments starting with `#` in the config files.

The script above can also be found as GitHub Gist at https://gist.github.com/yi-jiayu/5383cd268a1aa038e50c863f3d814784.

### Usage

I placed the script onto my path as `hotcorners` and made it executable.

I then saved my usual Hot Corners configuration and a modified configuration which uses the Command key as a modifier key for each corner into my config directory:

```console
$ cat .hotcorners/on
# this line should be ignored
wvous-bl-corner=2
wvous-bl-modifier=0
wvous-br-corner=4
wvous-br-modifier=0
wvous-tl-corner=3
wvous-tl-modifier=0
wvous-tr-corner=12
wvous-tr-modifier=0
$ cat .hotcorners/command 
wvous-bl-corner=2
wvous-bl-modifier=1048576
wvous-br-corner=4
wvous-br-modifier=1048576
wvous-tl-corner=3
wvous-tl-modifier=1048576
wvous-tr-corner=12
wvous-tr-modifier=1048576
```

Now, if I need to quickly disable my Hot Corners while pairing, I just run `hotcorners command` to apply the configuration requires Command to be pressed for the hot corners to activate, and `hotcorners on` to restore my usual configuration afterwards.

For the sake of completeness, I'm tempted to add a subcommand to my script to save the current Hot Corners configuration, or even write a more general script for configuring Hot Corners with more bells and whistles.
