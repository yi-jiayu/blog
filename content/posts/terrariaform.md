---
title: "Terrariaform"
subtitle: "Building a Terraria server with Terraform"
date: 2019-12-24T15:28:56+08:00
tags: ["terraform", "digitalocean", "terraria"]
---

I think [Terraria](https://terraria.org/) is a great game, and according to
Steam I've spent 572 hours of my life in it.

I'm eagerly awaiting the [1.4
update](http://terraria.org/news/re-logic-announces-terraria-journey-s-end-at-e3),
and when its out I want to run a server and start over with some friends.

While waiting, I thought I'd get a head start on the server setup. Rather than
manually launch a virtual machine from a cloud vendor's user interface and then
running some ad-hoc commands to experiment, I thought I'd automate things from
the beginning with [infrastructure as
code](https://en.wikipedia.org/wiki/Infrastructure_as_code) using
[Terraform](https://www.terraform.io/).

After about a day's work, I ended up with a Terraform project which deploys a
[DigitalOcean](https://www.digitalocean.com/) droplet with appropriate firewall
rules, then runs [TShock](https://tshock.co/xf/index.php) in
[screen](https://www.gnu.org/software/screen/) properly supervised by
[systemd](https://www.freedesktop.org/wiki/Software/systemd/). The entire setup
is fully automated---no action is required (other than a bit of waiting)
between running `terraform apply` and joining the server in-game.

The code and instructions can be found on GitHub:

{{< linkpreview title="yi-jiayu/terrariaform"
description="Terraform project to run a Terraria server"
url="https://github.com/yi-jiayu/terrariaform" >}}

Here's the obligatory disclaimer that things are still a little rough around
the edges. For one, many things are still hardcoded, such as droplet size and
world name.
