---
title: "Get notified when a process is preventing sleep on macOS"
date: 2018-12-25T12:34:09+08:00
image: "notification.png"
---

Twice in the past month, I came back to my (company-issued) MacBook after leaving it with the lid open overnight to find it refusing to start because its battery was completely drained, worrying me a little each time. I wondered why it wasn't going to sleep due to inactivity.

## Finding the culprit with the `pmset` command

Recently, I've been exploring various macOS command-line configuration tools such as `defaults` (see [my previous post](/2018/12/quickly-configuring-hot-corners-on-macos/)), `scutil`, `osascript` etc. I finally discovered why my Mac wasn't sleeping when expected after seeing the output of the `pmset` command:

```console
$ pmset -g
System-wide power settings:
Currently in use:
 standbydelaylow      10800
 standby              1
 womp                 1
 halfdim              1
 hibernatefile        /var/vm/sleepimage
 proximitywake        1
 powernap             1
 gpuswitch            2
 networkoversleep     0
 disksleep            10
 standbydelayhigh     86400
 sleep                1 (sleep prevented by Intel(R) Power Gadget)
 hibernatemode        3
 ttyskeepawake        1
 displaysleep         10
 tcpkeepalive         1
 highstandbythreshold 50
 acwake               0
 lidwake              1
```

According to the man page, `pmset` is a command which manipulates power management settings. The `-g` flag (with no argument) will display the settings currently in use. `pmset` will also reveal when there are processes interfering with any usual settings:

> Whenever processes override any system power settings, pmset will list those processes and their power assertions in -g and -g assertions.

From the output above, I could see that the [Intel(R) Power Gadget](https://software.intel.com/en-us/articles/intel-power-gadget-20) utility which I had been running to monitor my Mac's processor load and temperature was also preventing my system from sleeping! I hastily disabled it and hoped that my Mac would no longer drain its battery overnight again.

## It's not over yet

This morning, I came back to an even bigger surprise: my MacBook was still awake, and this time, even the screen was still on! I immediately checked `pmset -g` again:

```console
$ pmset -g
System-wide power settings:
Currently in use:
 standbydelaylow      10800
 standby              1
 womp                 1
 halfdim              1
 hibernatefile        /var/vm/sleepimage
 proximitywake        1
 powernap             1
 gpuswitch            2
 networkoversleep     0
 disksleep            10
 standbydelayhigh     86400
 sleep                1 (sleep prevented by screencaptureui)
 hibernatemode        3
 ttyskeepawake        1
 displaysleep         10 (display sleep prevented by screencaptureui)
 tcpkeepalive         1
 highstandbythreshold 50
 acwake               0
 lidwake              1
```

This time, there was a process called `screencaptureui` which was preventing not just sleep, but also display sleep!

Apparently, `screencaptureui` is the new screenshot tool you can activate with Cmd-Shift-5 introduced in macOS 10.14 Mojave. (Thinking back, I did have an issue the previous day when I was taking a screen recording but ran into a bit of trouble finding the stop button in the menu bar afterwards.)

Anyway, killing `screencaptureui` was a short-term fix, but the underlying issue was that it was hard to tell if there was any process preventing sleep (or display sleep) without manually checking `pmset`.

## A cron job to the rescue

### The script

I wrote a short script which checks the output of `pmset -g` for any processes preventing sleep, then triggers a Notification Centre notification if there are any:

```shell
#!/bin/sh -

sleep_blocker=$(pmset -g | grep -m1 "sleep prevented by" | sed -E 's/.+sleep prevented by (.+)\)$/\1/')
if [ ! -z "$sleep_blocker" ]; then
	osascript -e "display notification \"$sleep_blocker\" with title \"Sleep prevention warning\" subtitle \"The following processes are preventing sleep:\""
fi
```

For example, if Intel(R) Power Gadget is running, the following notification will be shown:

{{< figure src="notification.png" alt="Notification showing that Intel(R) Power Gadget is preventing sleep" class="nooutline" >}}

The script above can also be found at https://gist.github.com/yi-jiayu/f1a8f32d70a6a4c3b7dbd81589c40f0d.

### Adding it to cron

Running `crontab -e` opens your per-user crontab file in your preferred editor. I added the following line to it:

```crontab
*/30 * * * * /Users/yijiayu/sleep_prevention_warning.sh >/dev/null 2>&1
```

This registers my script to run every half an hour, using the cron expression I got from [here](https://crontab.guru/every-half-hour), and I tested it to work by leaving my Intel(R) Power Gadget open until the cron job triggered.

## Wrapping up

While writing this, I noticed that Chrome also prevents sleep quite often, although its never caused me any issues.

I'm mostly worried about additional noise from yet another source of notifications, but at this point in time it's too early to say. Ideally most of the time the script should be a no-op, since there shouldn't be any processes preventing sleep normally (right?).

Triggering notifications with `osacript` is pretty rudimentary, and doesn't let you register anything to happen when you click it, such as bringing up more detailed information from `pmset -g assertions`. Perhaps it would be an interesting project to write a full Cocoa menu bar application to monitor processes preventing the system from sleeping? (It would be ironic if such an application ended up preventing the system from sleeping though.)
