---
title: "Hosting your own web fonts"
subtitle: "Building Iosevka and serving it locally instead of using something from Google Fonts"
date: 2018-06-30 00:07:55+08:00
tags: ["fonts", "Iosevka", "CSS"]
---

I've replaced Source Code Pro with a variant of `Iosevka` as the monospace font for my blog theme! While Source
Code Pro [is available on Google Fonts](https://fonts.google.com/specimen/Source+Code+Pro) and could be included
from there with a single line of code, Iosevka wasn't, so a bit more work was needed.

I've been a fan of Iosevka and using it as my coding font since the day I first saw it appear on [Hacker
News](https://hn.algolia.com/?query=iosevka). It's noticeably narrower than most monospace fonts, but that was a
part of its appeal for me---at the very least, you can fit more code into the horizontal space on your screen.

Here's some Haskell code from [my
solution](https://github.com/yi-jiayu/haskell-advent-of-code/blob/master/src/day_22.hs#L83-L88) for [Advent of
Code 2015 day 22](http://adventofcode.com/2015/day/22):

```haskell
cast :: Spell -> GameState -> GameState
cast spell state = case spell of MISSILE -> castMissile state
                                 DRAIN -> castDrain state
                                 SHIELD -> castShield state
                                 POISON -> castPoison state
                                 RECHARGE -> castRecharge state
```

Iosevka is a highly customisable font. Besides the multiple pre-built variants you can download from [its GitHub
releases](https://github.com/be5invis/Iosevka/releases), you can also build it from source to exercise full control
over the many available customisation options. These range from options that affect the overall look and feel such
as ligations, serifs and spacing, to those that change how individual characters look---whether you want a double-
or single-storey `a` and `g`, whether the middle leg of the letter `m` should be shorter than the other two or
whether the number `3` should have a flat top.

You can see a visual representation of all the available styles and customisation options at the Iosevka GitHub
repository:

{{< linkpreview title="be5invis/Iosevka" description="Slender typeface for code, from code." url="https://github.com/be5invis/Iosevka" >}}

## Building Iosevka
Besides customisation, Iosevka also has to be built from source to get the `woff` and `woff2` web font formats.

### Dependencies
Building Iosevka requires a handful of dependencies. On my platform (Ubuntu 16.04 on WSL), some of these were
available from package management or as binary downloads, while the rest had to be built from source.

- `node`---installed using [`nvm`](https://github.com/creationix/nvm).
- `ttfautohint`---installed through `apt-get`.
- `otfcc`---built from source from [caryl/otfcc](https://github.com/caryll/otfcc). Required `premake5` to build,
which could be downloaded as a prebuilt binary from [here](https://premake.github.io/download.html).
- `sfnt2woff`---installed through `apt-get` from the `woff-tools` package.
- `woff2_compress`---built from source from [google/woff2](https://github.com/google/woff2).

After ensuring all the dependences have been added to your `$PATH`, clone the Iosevka repository, check out the
latest release (`v1.14.3` at time of writing) and install Node dependencies with `npm install` or `yarn`.

### Customisation
Building a custom variant of Iosevka involves two separate Makefile targets. `make custom-config` will generate
a configuration file which will be used by `make custom` or `make custom-web` to build the actual font files.

Since I was building the `ss05` (Fira Mono style) preset of Iosevka `v1.14.3`, I specified `set=ss05-1.14.3`. I
also only needed the book (regular) and bold weights:

```console
make custom-config set=ss05-1.14.3 design=ss05 weights='book bold'
make custom-web set=ss05-1.14.3
```

This populates the `dist/iosevka-ss05-1.14.3/` directory:
```console
$ tree dist/iosevka-ss05-1.14.3/
dist/iosevka-ss05-1.14.3/
├── ttf
│   ├── iosevka-ss05-1.14.3-bolditalic.ttf
│   ├── iosevka-ss05-1.14.3-boldoblique.ttf
│   ├── iosevka-ss05-1.14.3-bold.ttf
│   ├── iosevka-ss05-1.14.3-italic.ttf
│   ├── iosevka-ss05-1.14.3-oblique.ttf
│   └── iosevka-ss05-1.14.3-regular.ttf
├── woff
│   ├── iosevka-ss05-1.14.3-bolditalic.woff
│   ├── iosevka-ss05-1.14.3-boldoblique.woff
│   ├── iosevka-ss05-1.14.3-bold.woff
│   ├── iosevka-ss05-1.14.3-italic.woff
│   ├── iosevka-ss05-1.14.3-oblique.woff
│   └── iosevka-ss05-1.14.3-regular.woff
└── woff2
    ├── iosevka-ss05-1.14.3-bolditalic.woff2
    ├── iosevka-ss05-1.14.3-boldoblique.woff2
    ├── iosevka-ss05-1.14.3-bold.woff2
    ├── iosevka-ss05-1.14.3-italic.woff2
    ├── iosevka-ss05-1.14.3-oblique.woff2
    └── iosevka-ss05-1.14.3-regular.woff2

3 directories, 18 files
$
```

## Serving web fonts

The line of code you add from Google Fonts loads a CSS stylesheet which tells browsers where to find the fonts
you'll be using.

For example, including [two different weights of Source Code
Pro](https://fonts.google.com/selection?selection.family=Source+Code+Pro:400,700) from Google Fonts will load this
piece of CSS[^1] from [here](https://fonts.googleapis.com/css?family=Source+Code+Pro:400,700):

```css
/* latin-ext */
@font-face {
  font-family: 'Source Code Pro';
  font-style: normal;
  font-weight: 400;
  src: local('Source Code Pro'), local('SourceCodePro-Regular'), url(https://fonts.gstatic.com/s/sourcecodepro/v7/HI_SiYsKILxRpg3hIP6sJ7fM7PqlM-vWjMY.woff2) format('woff2');
  unicode-range: U+0100-024F, U+0259, U+1E00-1EFF, U+2020, U+20A0-20AB, U+20AD-20CF, U+2113, U+2C60-2C7F, U+A720-A7FF;
}
/* latin */
@font-face {
  font-family: 'Source Code Pro';
  font-style: normal;
  font-weight: 400;
  src: local('Source Code Pro'), local('SourceCodePro-Regular'), url(https://fonts.gstatic.com/s/sourcecodepro/v7/HI_SiYsKILxRpg3hIP6sJ7fM7PqlPevW.woff2) format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
/* latin-ext */
@font-face {
  font-family: 'Source Code Pro';
  font-style: normal;
  font-weight: 700;
  src: local('Source Code Pro Bold'), local('SourceCodePro-Bold'), url(https://fonts.gstatic.com/s/sourcecodepro/v7/HI_XiYsKILxRpg3hIP6sJ7fM7Pqths7Dvecq_mk.woff2) format('woff2');
  unicode-range: U+0100-024F, U+0259, U+1E00-1EFF, U+2020, U+20A0-20AB, U+20AD-20CF, U+2113, U+2C60-2C7F, U+A720-A7FF;
}
/* latin */
@font-face {
  font-family: 'Source Code Pro';
  font-style: normal;
  font-weight: 700;
  src: local('Source Code Pro Bold'), local('SourceCodePro-Bold'), url(https://fonts.gstatic.com/s/sourcecodepro/v7/HI_XiYsKILxRpg3hIP6sJ7fM7Pqths7Ds-cq.woff2) format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
```

Each `@font-face` block specifies, for a specific font family, style and weight, where to load the font from. The
`local()` function tries to load a font installed locally on the machine, while the `url()` points to where it
can be downloaded from online, and in what format.

Some other useful attributes for further optimising font delivery are `font-display`, which determines how long
to wait for a font to load or whether to replace it with another font instead, and `unicode-range` as seen above,
which specifies exactly which characters to be displayed in a particular font.

### Loading our own fonts

To serve our own web fonts locally, we just need to include our own `@font-face` declarations which point to our
locally hosted font files.

I only planned to need two different styles, normal 400 for most places and a bold version for headings. Building
Iosevka for web generated `woff2`, `woff` and `ttf` versions, so I included them all:

```css
@@font-face {
   font-family: 'Iosevka';
   font-style: normal;
   font-weight: 400;
   src: url('/fonts/iosevka-ss05-1.14.3-regular.woff2') format('woff2'), url('/fonts/iosevka-ss05-1.14.3-regular.woff') format('woff'), url('/fonts/iosevka-ss05-1.14.3-regular.ttf') format('truetype');
 }
 
 @font-face {
   font-family: 'Iosevka';
   font-style: normal;
   font-weight: 700;
   src: url('/fonts/iosevka-ss05-1.14.3-bold.woff2') format('woff2'), url('/fonts/iosevka-ss05-1.14.3-bold.woff') format('woff'), url('/fonts/iosevka-ss05-1.14.3-bold.ttf') format('truetype');
 }

```

(I chose to include the version number in the path to hopefully avoid issues with caching when I decided to update
to a newer version of Iosevka.)

Afterwards, just load your own `@font-face` declarations as you would any other CSS. I included it in [its own
CSS file](/css/iosevka.css) and included it with:
```html
<link rel="stylesheet" href="/css/iosevka.css">
```

## Further reading

I found this article by Twitter engineer Mike Solomon to be very informative both while setting things up myself
and writing this post afterwards:

{{< linkpreview title="Host your own web fonts" description="Mike Solomon" url="https://msol.io/blog/tech/host-your-own-web-fonts/" >}}

[^1]: According to [CSS-Tricks](https://css-tricks.com/dont-just-copy-the-font-face-out-of-google-fonts-urls/), Google Fonts is pretty fancy and does things like varying the exact contents of the loaded CSS depending on browser. For example, `woff2` is a relatively new format and `woff2` URLs may not be included when requested by an older browser which doesn't support it.
