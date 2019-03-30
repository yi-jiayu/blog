---
title: "Rotom Pokédex bot"
slug: "rotom-pokedex-bot"
subtitle: "A Telegram bot for looking up Pokémon-related things"
date: 2019-03-30 23:18:24+08:00
draft: true
---

Lately, I've been playing through some of the older Pokémon games (Pokémon Platinum and Pokémon
White), and built a Telegram bot for quickly looking up in-game things like Pokémon weaknesses or
ability effects.

Send Rotom Pokédex bot ([@rotom_pokedex_bot](https://t.me/rotom_pokedex_bot) on Telegram) a Pokémon
name or number to get a quick summary about it:

{{< figure class="nooutline" src="summary.png" caption="Turtwig, the Tiny Leaf Pokémon" >}}

The inline buttons let you navigate to more detailed information:

{{< figure src="stats-cropped.png" caption="Turtwig's base stats" >}}

{{< figure src="evolutions-cropped.png" caption="Turtwig's evolutions" >}}

{{< figure src="locations-cropped.png" caption="Where to catch Dratini" >}}

Rotom Pokédex bot also knows about abilities, items and moves:

{{< figure src="item-cropped.png" caption="Timer ball" >}}

{{< figure src="ability-cropped.png" caption="Pixilate" >}}

{{< figure src="move-cropped.png" caption="Grass knot" >}}

Using inline queries, you can search and send Pokédex entries to other chats as well:

{{< figure src="inline-cropped.png" alt="An inline query for the term \"magne\" showing results for Magnet, Magby, Rage and Mime Jr." >}}

Check out Rotom Pokédex bot on Telegram at [@rotom_pokedex_bot](https://t.me/rotom_pokedex_bot)!

## Background

A long time ago, I wrote a [series][1] [of][2] [posts][3] about the [Telegram Bot API][8] on Medium.
It walked readers through the process of creating an [inline Telegram bot][4] running on [Google
Cloud Functions][5] which served Pokédex data from a [JSON document][6] on the [Pokémon website][7].

That was the [original Rotom Pokédex bot][9]. Limited by the data and an arbitrary 100 LOC limit I set to
keep it simple, it was mostly a novelty, its usefulness limited to finding out what a particular
Pokémon looked like. The entire exercise was really just because I was looking for things to write
about.

However, actually playing the Pokémon games gave me a chance to dogfood my own creation, and I
decided to ~~rewrite~~ evolve it to actually be useful.

## Post-evolution

Rotom Pokédex bot is now powered by the excellent `pokedex` library by Veekun:

{{< linkpreview title="veekun/pokedex" description="more than you ever wanted to know about Pokémon"
url="https://github.com/veekun/pokedex" >}}

The library is used used by [Veekun's Pokédex][11] and many others, as well as projects like
[PokéAPI](https://pokeapi.co/).

Rotom Pokédex bot itself is a simple Python 3 webapp using [Tornado][12]:

{{< linkpreview title="yi-jiayu/rotom-pokedex-bot-2"
description="Rotom Pokédex bot evolved"
url="https://github.com/yi-jiayu/rotom-pokedex-bot-2" >}}

Although the documentation for Veekun's pokedex library specifically mentions that it doesn't work
with Python 3, I was pleasantly surprised to find out that that wasn't the case---it's been working
out of the box for me on Python 3 with no issues so far. All credit should go to the pokedex team,
which has been [actively fixing Python 3 compatibility issues][13].

Rotom Pokédex bot is still a work in progress, and there are many features I haven't gotten around
to implementing. For example, I often want to find out what moves a Pokémon can learn and at what
level, so it would be nice to be able to view movesets with Rotom Pokédex bot (I've been
experimenting a bit with this, and presentation is an issue on narrower screens).

Anyway, do give Rotom Pokédex bot a try, and support the project on GitHub if you're interested! I'm
also planning a follow up post on a Caddy middleware I wrote to collect usage statistics for Rotom
Pokédex bot without having to implement the logic in the application code itself.

[1]: https://chatbotslife.com/introduction-to-the-telegram-bot-api-part-1-2ae36f7b30a4
[2]: https://chatbotslife.com/introduction-to-the-telegram-bot-api-part-2-9d443bf8f17a
[3]: https://chatbotslife.com/introduction-to-the-telegram-bot-api-part-3-d09495fe387d
[4]: https://core.telegram.org/bots/inline
[5]: https://cloud.google.com/functions/
[6]: https://www.pokemon.com/us/api/pokedex/kalos
[7]: https://www.pokemon.com/us/pokedex/
[8]: https://core.telegram.org/bots/api
[9]: https://gist.github.com/yi-jiayu/6a9a84ff8d3ef754d96f6752cf459905#file-index-js
[10]: https://github.com/veekun/pokedex/wiki/Pokedexen
[11]: https://veekun.com/dex
[12]: https://www.tornadoweb.org/en/stable/
[13]: https://github.com/veekun/pokedex/search?q=python+3&unscoped_q=python+3&type=Commits

