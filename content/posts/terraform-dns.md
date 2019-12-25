---
title: "Managing personal DNS records with Terraform"
subtitle: "Small ways to use infrastructure-as-code in your daily life"
date: 2019-12-25T13:47:39+08:00
---

If you own a domain or run a website, chances are you had to set some DNS
records before.

The usual way of doing this involves logging in to your account on your
registrar's website and clicking your way through their UI. A more sophisticated
procedure might be to change your nameservers to a service which lets you set
DNS records programatically, for example using [AWS Route
53](https://aws.amazon.com/route53/) and the AWS CLI.

This was exactly the path I took up till now---initially I used the
[Namecheap](https://www.namecheap.com/) website directly, then I tried to
automate things using the [Namecheap
API](https://www.namecheap.com/support/api/intro/) but my request for production
environment access was never granted, so I changed my nameservers to Route 53
instead. I even created a service to set DNS records through a Telegram bot.

However, all these methods are still ad-hoc and mean that you have to check with
your DNS provider to know what records you have already set. It's easy to lose
track of what records you have set at the moment and forget to unset records
that are no longer required, leaving you vulnerable to subdomain takeovers.

Infrastructure-as-code (IaC) is a way to tackle some of these issues, allowing you to
maintain a canonical list of all the DNS records you want and have a way to
automatically keep your live DNS records in sync. It's easier to manage such an
inventory in a single place, especially if you have records spread across
multiple DNS providers.

[Terraform](https://www.terraform.io/) is a commonly-used tool for practicing
IaC, and it's easy to use it to manage your DNS records. Here's what a part of
my Terraform project for managing my DNS records looks like:

```
resource "aws_route53_zone" "jiayu_dot_co" {
  name = "jiayu.co"
}

# Point apex domain at Netlify, which is configured to to redirect jiayu.co to
# www.jiayu.co.
resource "aws_route53_record" "apex" {
  zone_id = aws_route53_zone.jiayu_dot_co.zone_id
  name    = "jiayu.co"
  type    = "A"
  ttl     = 86400
  records = ["104.198.14.52"]
}

resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.jiayu_dot_co.zone_id
  name    = "www.jiayu.co"
  type    = "CNAME"
  ttl     = 86400
  records = ["frame-maker-mole-38374.netlify.com."]
}

resource "aws_route53_record" "blog" {
  zone_id = aws_route53_zone.jiayu_dot_co.zone_id
  name    = "blog.jiayu.co"
  type    = "CNAME"
  ttl     = 86400
  records = ["zealous-perlman-cb8cb5.netlify.com."]
}
```

Here, I've declared an AWS Route 53 zone for the domain `jiayu.co` and some
records in that zone for my landing page and blog. I can even add comments to
remind me why I have certain records set---when I was migrating my existing
records over, I couldn't remember why I had a random A record set on the apex
domain.

Once you have your DNS records defined in Terraform (and your relevant Terraform
providers set up), a single `terraform apply` is all that's needed to take them
live. Going forward, you just need to edit your DNS records in a single place
and apply the changes again.
