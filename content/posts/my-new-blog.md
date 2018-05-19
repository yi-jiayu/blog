---
title: "My New Blog"
subtitle: "Moving away from Medium (but not too far)"
date: 2018-04-29T15:32:50+08:00
tags: ["Blogging", "Medium"]
---

## One year ago
In June last year, I started a blog on Medium and wrote my first post on the highly public Lee family feud over their family home which was taking place around that time. It was something I decided to write on a whim that day, after a friend showed me the news that morning.

{{< linkpreview "Sentiment Analysis of Comments on LHL’s Facebook Page" 
"Learning how to use the Facebook Graph API and Google Cloud Natural Language API" 
"https://medium.com/google-cloud/sentiment-analysis-of-comments-on-lhls-facebook-page-9db8b3a60eb3" >}}
{{< caption "My first blog post" >}}

Despite the provocative topic, it was really just a post about how to download comments from the Facebook Graph API and perform some sentiment analysis using the Google Cloud Natural Language API. Nevertheless, I was surprised by the reception my post received: Tech in Asia invited me to [republish it on their site](https://www.techinasia.com/feel-lee-hsien-loong-code), [Asiaone](http://www.asiaone.com/singapore/computer-science-student-analyses-pm-lees-facebook-comments-public-feud-and-what-he-got) and [Lianhe Zaobao](http://www.zaobao.com.sg/special/report/singapore/38oxley/feature/story20170630-775425) ran articles about me, it was added to the official [Google Cloud Platform publication on Medium](https://medium.com/google-cloud), and it even attracted [3 pages of discussion on EDMW](https://archive.is/4gxUt).

Naturally, not all the feedback was positive and I felt that my article was being taken too seriously as a commentary on our PM's approval rating rather than a simple technical experiment.

> Stupid SUTD student
> 
> joker. do analysis and then says "by no means a source of objective views"
> 
> He is an opportunist wanting to show off his skillset, but highlighted his own shortcoming when he chose wrong source of data ~

{{< caption "Some choice comments I found" >}}

However, I couldn't help but feel slightly flattered by the attention and also thought that writing was quite fun, having been thinking of starting a blog for some time already as a way to build my brand, so I continued to write on Medium.

## Writing on Medium
### My own experience
Medium's site was clean and neat and I especially liked the writing experience in Medium's editor. In addition, tags and publications were a great help in attracting attention to your posts. I was also influenced by other organisations which decided to maintain a presence on Medium such as [Google Cloud Platform](https://medium.com/google-cloud) (which has accepted a handful of my posts) [Data.gov.sg](https://blog.data.gov.sg/) and [Netflix Engineering](https://medium.com/netflix-techblog).

### A different perspective
If you try searching online for "should I write on Medium" you'll find no shortage of arguments both for and against it. However, what really struck me was one comment I came across on [Hacker News](https://news.ycombinator.com/item?id=16672515) (emphasis mine):

> I rarely click medium.com links, or any of their associated domains, hackernoon.com, etc. 

> The website sucks. It's crippled without JS, and squirmy and laggy when JS is enabled. Either way, it's very heavy on the network pipe.
  
> Medium is in business of driving traffic, which means the content is likely to be mediocre.
  
> It's content that is looking for an audience, as my comment's sibling states. Which means that it is probably not that compelling, otherwise the audience would find it.
  
> **It's written by someone who can't be bothered to set up their own website without all of these mis-features, nor understands the importance of doing so.**
  
> **And for all of these reasons, the opinion of someone who publishes on Medium is worth a lot less to me.**
  
> The same goes for businessinsider.com, wsj.com, patch.com, nymag.com and all those other shitty sites that make me regret visiting them the moment I arrive. There can't be anything relevant enough on there that I can't live without. Just a big waste of my time and network resources.
  
> I've blacklisted them in my hosts file, and haven't looked back.

It's true that there are some recurring complaints about various aspects of Medium that I agree with, such as the persistent "Open in app" button which appears over the article you are reading on mobile browsers. However, the parts of this comment which hit me the hardest (in bold) were not about Medium directly.

As someone who does take pride in the various applications I have set up myself, it hurt me a little to be called "someone who can't be bothered to set up their own website". While there are very real reasons to opt for managed services over self-hosted infrastructure, I'm not sure if they apply to a choice of Medium or Wordpress.com or Blogger or self-hosted Wordpress or a simple web application on a $5/mo VPS or a static site on GitHub pages/AWS S3/Netlify for a personal blog. Personally, I had always wanted to host my own blog as a static site _just because I could_, but I never got around to doing it.

The following line, which stated that "the opinion of someone who publishes on Medium is worth a lot less to me", also made me realise that first impressions are important. They're also fickle, and sometimes you can't really control the first impression someone gets of you, whether in real life or online---maybe they had a bad experience with someone else who shared the same name as you before, or you're just writing on a platform that they don't like.

## A simple fix
In my case though, there was a simple solution: it's something I've been wanting to do for a while, and it might improve my first impression on some people, so I finally set up my own blog.

### Tech specs
- I previously set up the subdomain `blog.jiayu.co` as a temporary HTTP 302 redirect to my Medium profile page before I created this site.
- It's generated by [Hugo](https://gohugo.io/), a static site generator written in [Go](https://golang.org/), a language I am quite partial to.
- The theme is entirely written from from the ground up by myself, (hopefully not too strongly) inspired by Medium's own design.
- It's hosted on [Netlify](https://www.netlify.com/), which automatically builds and serves the generated files whenever I commit to my [GitHub repository](https://github.com/yi-jiayu/blog) so I don't have to check them into version control.
- HTTPS is enabled with a [Let's Encrypt](https://letsencrypt.org/) certificate [automatically provisioned](https://www.netlify.com/docs/ssl/) by Netlify. Netlify also helps to redirect from HTTP to HTTPS and set the `Strict Transport Security` header, and serves content over HTTP2 automatically once HTTPS is enabled.

## Going forward
I'm happy with my new blog, but I have also benefited greatly from Medium's ability to drive traffic. To have my cake and eat it too, I intend to continue publishing on Medium using their "Import post" feature to cross-post my new posts there as well, starting with this one.