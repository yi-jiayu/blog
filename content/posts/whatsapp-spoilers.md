---
title: "WhatsApp Spoilers"
subtitle: "Fun with zero-width spaces in WhatsApp"
date: 2018-05-02T21:57:51+08:00
lastmod: 2018-05-03T10:11:10+08:00
tags: ["WhatsApp", "JavaScript"]
---

On Reddit or other forums, you'll often come across spoiler tags <span class="spoiler">which may look a bit like this</span>. They're used to discuss spoilers while protecting other readers who do not wish to be spoiled by requiring them to actively interact with the spoiler tag to view its contents.

Conspicuously, spoiler tags are a missing feature in most instant messaging applications, even though it seems just as easy to get accidentally spoiled by a single careless message in a group conversation:

{{< figure src="/posts/images/spoiler.png" alt="Getting spoiled in a group chat" caption="Oops" link="/posts/images/spoiler.png" >}}

Fortunately, in WhatsApp at least, we can emulate a spoiler tag by taking advantage of the way long messages are hidden behind a "Read more" fold:

{{< figure src="/posts/images/spoiler-tag.png" alt="A spoiler tag in WhatsApp" link="/posts/images/spoiler-tag.png" >}}

Although it doesn't look like it, the message above is at least 4000 characters long. Between the spoiler warning and the actual text are 4000 zero-width spaces: `U+200B ZERO WIDTH SPACE` in Unicode or `&#8203;` in HTML, enough to cause the displayed message to be truncated.

## The inspiration
{{< figure src="/posts/images/whatsapp-needs-a-spoiler-tag.png" alt="Discussion about spoiler tags in WhatsApp" link="/posts/images/whatsapp-needs-a-spoiler-tag.png" >}}

My friend Ji An and I got the idea when we were discussing our solutions to last year's [Advent of Code](https://adventofcode.com/)---we were doing so in a WhatsApp group for our shared leaderboard, but we didn't want to spoil the experience for those of us who had yet to solve the problem being discussed.

## Demo
Resourceful as he was, Ji An quickly demonstrated that it was possible. It was a bit inconvenient to manually insert a bunch of zero-width spaces when composing your message, so we made a simple demo website to generate a WhatsApp spoiler message which you could copy and paste:

{{< linkpreview "WhatsApp Spoilers" "Hide content behind a \"Read more\" button in a WhatsApp message." "https://yi-jiayu.github.io/whatsapp-spoilers/" >}} 

### How it works
```javascript
function createSpoilerMessage() {
  var warning = document.getElementById('warning').value;
  var content = document.getElementById('content').value;
  var message = warning + ' ' + '\u200B'.repeat(4000) + content;
  document.getElementById('message').value = message;
}
```
{{< caption "What's wrong with inline JavaScript?" >}}

After setting a spoiler warning to be shown before the spoiler (`warning`) and the actual content of the spoiler (`content`), we add 4000 zero-width spaces (for safety---around 3100 seems to be enough on WhatsApp Web, but more are needed on mobile) between the two and display the result for the user to copy.

### Caveats
There are some side effects with this hack though.

Firstly, adding 4000 zero-width spaces makes the message length more than 4000 characters long. In fact, because a zero-width space is encoded under UTF-8 as 3 bytes (`E2 80 8B`), a message with a spoiler tag may actually be larger than 12 kB in size! It actually causes some unexpected behavior in WhatsApp, such as truncated message histories or just a general slowdown.

Also, on WhatsApp Web, the spoiler still appears in the chat preview:

{{< figure src="/posts/images/movie-spoilers.png" alt="Spoiler contents are visible in the chat preview" link="/posts/images/movie-spoilers.png" >}}

(At least it seems to be hidden properly in web notifications, and both mobile chat previews and notifications.)

Fortunately, a workaround for this is just to make the spoiler banner long enough to overflow the chat preview:

{{< figure src="/posts/images/movie-spoilers-hidden.png" alt="Spoiler contents pushed out of chat preview" link="/posts/images/movie-spoilers-hidden.png" >}}

## Final thoughts
Although this was something we did at the end of last year, since _Avengers:&nbsp;Infinity War_ was just released in cinemas recently and it's relatively plot-heavy for a Marvel film, I thought it was a good time to write about this.

I'm not actually sure if there are any instant messaging apps which support tags natively, although while I was looking around I found some people who built bots for Slack and Telegram to hide spoilers:

- [Hide it Bot](https://github.com/erpheus/hideit-bot) (Telegram)
- [slack-spoilers](https://github.com/indspenceable/slack-spoilers) (Slack)
- [Spoiler](http://spoiler.fountstudio.com/) (Slack, [blog post](https://blog.fountstudio.com/spoiler-a-slack-app-to-prevent-spoilers-de634bc7497d))
