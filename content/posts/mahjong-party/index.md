---
title: "Mahjong Party"
date: 2020-09-22T18:41:33+08:00
subtitle: "Looking for beta testers!"
---

I recently built [Mahjong Party](https://mahjong.party/), an online multiplayer
Singaporean mahjong game.

Here's a screenshot of a game in progress:

{{< figure src="mahjong-party.png" alt="Screenshot of a mahjong party game in progress" >}}

If you're interested, you can head over to https://mahjong.party/ and give the
tutorial a try, or jump into a game immediately with some bots!

The rest of this post will just be some thoughts I had while working on this
project.

## Miscellaneous thoughts

This wasn't my first attempt at building a multiplayer game online, I previously
tried creating a [bughouse](https://en.wikipedia.org/wiki/Bughouse_chess)
xiangqi game, but gave up after running into challenges around UI design and
backend state management.

Rather than websockets, Mahjong Party uses a combination of [server-sent
events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) and
standard HTTP requests for client-server communication.

I went with React for the frontend because I'm not familiar with frontend
development at all and wanted to stick to a relatively boring technology.

Regarding online Singaporean mahjong, I realised I was late to the party---this
came out during the circuit breaker and seems to have been pretty well received:

{{< linkpreview title="Created a website to play Singaporean style Mahjong with friends!"
description="r/singapore"
url="https://www.reddit.com/r/singapore/comments/gk70o5/created_a_website_to_play_singaporean_style/" >}}

## Wrap up

Once again, do check out Mahjong Party and invite your friends to play too if
you are interested! I'd really appreciate it. Here's the link again:

{{< linkpreview title="Mahjong Party"
description="Play Singaporean mahjong online with friends"
url="https://mahjong.party/" >}}

It's not finished yet, but definitely in a playable state and I thought that it
would be good to get some user testing in before I continue development. You can
reach out to me directly or leave feedback on the GitHub repository:

{{< linkpreview title="yi-jiayu/mahjong-party"
description="Play Singaporean mahjong online with friends"
url="https://github.com/yi-jiayu/mahjong-party" >}}
