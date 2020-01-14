---
title: "Terraform Provider for Telegram"
subtitle: "Manage Telegram bot webhooks and more with Terraform"
date: 2020-01-14T18:01:58+08:00
tags: ["telegram", "terraform"]
---

Over the weekend, I wrote a [custom Terraform
provider](https://www.terraform.io/docs/extend/writing-custom-providers.html)
for Telegram:

{{< linkpreview title="yi-jiayu/terraform-provider-telegram"
description="Terraform provider for Telegram"
url="https://github.com/yi-jiayu/terraform-provider-telegram" >}}

My main goal was to have first-class Terraform support for setting a [bot
webhook](https://core.telegram.org/bots/api#setwebhook). Right now, you can
automate all the infrastructure for a Telegram bot except for setting the
webhook, which has to be done manually or with a workaround like using the
[`local-exec`](https://www.terraform.io/docs/provisioners/local-exec.html)
provisioner.

With the Terraform provider for Telegram, a bot webhook can be declared as a
Terraform resource:

```hcl
resource "telegram_bot_webhook" "example" {
  url = "https://www.example.com/webhook"
}
```

Creating, updating and deleting a bot webhook works as expected.

Besides setting a bot webhook, there is also a Telegram bot data source for
getting details about the currently-authenticated Telegram bot:

```hcl
data "telegram_bot" "example" {}

output "bot_link" {
  value = "https://t.me/${data.telegram_bot.example.username}"
}
```

## Getting started

Since the Terraform provider for Telegram is a third-party plugin, you need to
manually install it by dropping its executable into your user plugins directory.
You can find precompiled binaries for your OS on the [releases
page](https://github.com/yi-jiayu/terraform-provider-telegram/releases), or
build it from source with a Go toolchain (Go 1.13 or newer).

The user plugins directory is located at `%APPDATA%\terraform.d\plugins` on
Windows, and `~/.terraform.d/plugins` on all other systems. See the [Terraform
documentation on third-party
plugins](https://www.terraform.io/docs/configuration/providers.html#third-party-plugins)
for more information.

Once you've installed the plugin and added some Telegram resources, you'll need
to run `terraform init` again for Terraform to pick up the new provider before
you can run `terraform plan` or `terraform apply`.

## Support

If you're already using Terraform in your Telegram bot projects, give the
Terraform provider for Telegram a try! If you aren't already practising
infrastructure as code, it's a good opportunity to get started! Contributions
on GitHub are welcome.

I've also applied to have the Terraform provider for Telegram listed on the
[Terraform community
providers](https://www.terraform.io/docs/providers/type/community-index.html)
page.
