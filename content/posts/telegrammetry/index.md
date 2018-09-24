---
title: "Telegrammetry: Stats for Telegram"
subtitle: "Get insights into your Telegram activity"
image: "dashboard.png"
date: 2018-09-24T16:06:17+08:00
---

Last week, I set up a short pipeline comprised of a Python script which received updates from the Telegram API and
logged them, [Filebeat](https://www.elastic.co/products/beats/filebeat) to ingest the output into
[Elasticsearch](https://www.elastic.co/products/elasticsearch), and a [Kibana](https://www.elastic.co/products/kibana)
dashboard to visualise the data. Here's what the dashboard looks like:

{{< figure src="dashboard.png" alt="A Kibana dashboard showing Telegram activity" link="dashboard.png" >}}

I've decided to call this project Telegrammetry---combining Telegram and telemetry---and it can be found on GitHub (in a
very rough state right now):

{{< linkpreview title="yi-jiayu/telegrammetry" description="Stats for Telegram"
url="https://github.com/yi-jiayu/telegrammetry" >}}

## Background

Even before my [chat-analytics](https://github.com/yi-jiayu/chat-analytics) project for WhatsApp more than 2 years ago,
I've always thought there were a lot of interesting insights that could be found in our conversation histories, random
facts about ourselves purely for personal consumption rather than commercial benefit.

When I experimented with the Telegram API and used it to find my most active Telegram chats in [Introduction to the
Telegram API](https://towardsdatascience.com/introduction-to-the-telegram-api-b0cd220dbed2), I remember being surprised
by some of the results. However, that experiment didn't reflect how the data changed over time---for example, I
discovered one chat that was much higher in the rankings than I expected, which was even more surprising because I also
knew that I hadn't spoken much with that person for some time.

## Features and limitations

Telegrammetry provides real-time insights and time-series analysis of your activity on Telegram by recording all
incoming and outgoing Telegram messages and their timestamps.

Out of simplicity, Telegrammetry only counts new messages, although there's no reason why older messages can't be
extracted using the Telegram API and included as well.

Telegrammetry does not record all the fields of a Telegram message, although most of the important ones are there
including the message text and media types. One specific area that needs improvement is sticker support---stickers are
sent as `MediaTypeDocument` and only recorded as such, discarding sticker-specific data such as the sticker pack, emoji
and the sticker itself. Given how common sticker usage is in Telegram, being able to analyse sticker usage would be
good.

At the moment, Telegrammetry also does not perform any analysis on message text. Message sentiment and emoji usage
statistics would be nice to have.

## See also

- [chat-analytics](https://github.com/yi-jiayu/chat-analytics), an old project of mine featuring a limited set of
WhatsApp chat visualisations.
- [Introduction to the Telegram
API](https://towardsdatascience.com/introduction-to-the-telegram-api-b0cd220dbed2) and [Automatic replies for
Telegram](https://medium.com/@jiayu./automatic-replies-for-telegram-85075f28321), some articles I wrote previously about
the Telegram API
- [Chatilizer](https://chatilyzer.com/) ([blog
post](https://medium.com/@danielsternlicht/chatilyzer-a-whatsapp-chat-analyzer-visualization-tool-cbd1cde76ae2)),
another project I came across for analysing your WhatsApp conversations. (Seems to be doing the analysis on the client
side, although it doesn't seem to be open source.)
- [Telegram Analytics](https://tgstat.com/), a website displaying
various statistics such as total subscribers, subscriber growth, post reach etc. for many public Telegram channels.
- [Combot](https://combot.org/), a Telegram bot for helping you manage your chats, providing analytics and moderation.
