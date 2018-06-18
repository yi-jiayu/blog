---
title: "A compact representation of the SUTD logo"
slug: "a-minimal-representation-of-the-sutd-logo"
subtitle: "Playing with PBM images and ImageMagick"
date: 2018-06-17T15:41:50+08:00
tags: ["graphics", "ImageMagick", "SUTD", "Logo"]
---

I've always been a fan of geometric patterns. When I was younger I used to draw [Sierpinski
triangles](https://en.wikipedia.org/wiki/Sierpinski_triangle) in class after a classmate (hi Marken) introduced
them to me. I was fascinated by how hexagons tessellated after playing and designing maps for [The Battle for
Wesnoth](https://www.wesnoth.org/), an open-source, turn-based strategy game which plays out on a hex grid. I
still want to buy [this Ikea rug](https://www.ikea.com/sg/en/catalog/products/20260520/).

## The SUTD logo
As a [Singapore University of Technology and Design](https://sutd.edu.sg/) graduate, the university's logo also
intrigued me greatly:

{{< figure src="/images/SUTD-larger.png" alt="SUTD logo in black on white" link="/images/SUTD-larger.png" >}}

According to the [SUTD identity
guidelines](http://root.sutd.edu.sg/wp-content/themes/root/guidelinesandforms/SUTD_Identity_Guidelines.pdf),

> The simple, clean lines with no enclosures  symbolize an open culture. The squarish shape draws inspiration
from a  Chinese seal (the East), a microchip (technology) and a maze (problem  solving). Not least, the alphabets
(the West) reflect a University that  transcends national borders, cultures and economies (interactive).

But what I like most about it is its strictly defined, regular structure---all uniform spacing and right angles
with no troublesome angles or curves or colours---each letter is just a two-colouring of a 9x9 grid, a total of
81 bits of information.

For example, we can represent the letter S with 1s and 0s like this, and you can still discern it:

```
1 1 1 1 1 1 1 1 1
1 0 0 0 0 0 0 0 0
1 0 1 1 1 1 1 1 1
1 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 1
0 0 0 0 0 0 0 0 1
1 1 1 1 1 1 1 0 1
0 0 0 0 0 0 0 0 1
1 1 1 1 1 1 1 1 1
```

Since a compact encoding like this is enough to fully define the SUTD logo, we should be able to generate the more
mainstream versions of it from from this as well, with no loss of information.

## The PBM file format
Our textual representation of the letter S above is a bitmap of a binary image, with 1s representing filled pixels
and 0s representing blank pixels.

The [PBM (Portable Bit Map) image format](http://netpbm.sourceforge.net/doc/pbm.html) from the
[Netpbm](http://netpbm.sourceforge.net/) family of image formats is one of the simplest ways to represent such an
image. It encodes the dimensions and the state of each pixel in a binary image.

PBM and its sister formats [PGM (Portable Gray Map)](http://netpbm.sourceforge.net/doc/pgm.html) and [PPM (Portable
Pixel Map)](http://netpbm.sourceforge.net/doc/ppm.html) have a "raw" binary format as well as a "plain" format. While
binary formats represent pixels with individual bits, the plain formats use ASCII-encoded decimal numbers and are
much easier to read, although described as "even more simplistic, more lavishly wasteful of space" than the already
"expensive and not expressive enough" binary formats.

Here's our letter S in plain PBM format:

```
P1
9 9
1 1 1 1 1 1 1 1 1
1 0 0 0 0 0 0 0 0
1 0 1 1 1 1 1 1 1
1 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 1
0 0 0 0 0 0 0 0 1
1 1 1 1 1 1 1 0 1
0 0 0 0 0 0 0 0 1
1 1 1 1 1 1 1 1 1
```

It's almost the same as our previous representation, except we need to include the PBM header which comprises the
magic bytes `P1`, indicating that this is a plain PBM image, and `9 9`, the width and height of the image respectively.

We can compose the textual representations of each letter together to get the full SUTD logo in PBM format (remember
to update the image dimensions):

[SUTD.pbm](/files/SUTD.pbm)
```
P1
39 9
1 1 1 1 1 1 1 1 1 0 1 0 1 0 1 0 1 0 1 0 1 1 1 1 1 1 1 1 1 0 1 1 1 1 1 1 1 1 1
1 0 0 0 0 0 0 0 0 0 1 0 1 0 1 0 1 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1
1 0 1 1 1 1 1 1 1 0 1 0 1 0 1 0 1 0 1 0 1 1 1 0 1 0 1 1 1 0 1 1 1 1 1 1 1 0 1
1 0 0 0 0 0 0 0 0 0 1 0 1 0 1 0 1 0 1 0 0 0 1 0 1 0 1 0 0 0 0 0 0 0 0 0 1 0 1
1 1 1 1 1 1 1 1 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1 1 1 1 1 0 1 0 1
0 0 0 0 0 0 0 0 1 0 1 0 1 0 0 0 1 0 1 0 1 0 1 0 1 0 1 0 1 0 0 0 0 0 0 0 1 0 1
1 1 1 1 1 1 1 0 1 0 1 0 1 1 1 1 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1 1 1 1 1 1 1 0 1
0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 1 0 1 0 1 0 1 0 1 0 1 0 0 0 0 0 0 0 0 0 1
1 1 1 1 1 1 1 1 1 0 1 1 1 1 1 1 1 1 1 0 1 0 1 0 1 0 1 0 1 0 1 1 1 1 1 1 1 1 1
```

In a similar way we can obtain the [vertical](/files/SUTD-vertical.pbm) and [seal](/files/SUTD-seal.pbm) variants
of the SUTD logo in PBM format (apparently the seal variant is supposed to be reserved for official documents though).

## Exporting to other formats

Now that we have the SUTD logo in PBM format, we can use [ImageMagick's
`convert`](https://www.imagemagick.org/script/convert.php) tool to convert it to other formats.

Let's convert it to PNG using `convert SUTD.pbm SUTD.png`:

{{< figure src="/images/SUTD.png" alt="36x9 pixel SUTD logo in black on white background" link="/images/SUTD.png" >}}

That's an actual size, 39x9 PNG image.

Let's make it 10 times bigger using the `-scale` resize operator:

```
convert SUTD.pbm -scale 1000% SUTD-larger.png
```

{{< figure src="/images/SUTD-larger.png" alt="SUTD logo in black with white background" class="transparency-grid" link="/images/SUTD-larger.png" >}}

(The transparency grid is not part of the image---it's just there to make the image stand out against the white
background.)

It's important to ensure you pass a multiple of either 100% or the original image dimensions to the `-scale`
operator to ensure pixel-perfect scaling.

We can also make the white parts of the image transparent:

```
convert SUTD.pbm -scale 1000% -transparent white SUTD-transparent.png
```

{{< figure src="/images/SUTD-transparent.png" alt="SUTD logo in black with transparent background" class="transparency-grid" link="/images/SUTD-transparent.png" >}}

Finally, we can generate the white variant of the SUTD logo by inverting the colours and making black transparent
instead:

```
convert SUTD.pbm -scale 1000% -negate -transparent black SUTD-white-transparent.png
```

{{< figure src="/images/SUTD-white-transparent.png" alt="SUTD logo in white with transparent background" class="transparency-grid" link="/images/SUTD-white-transparent.png" >}}

Next time you need to use the SUTD logo, consider rendering it yourself instead of searching for it online (although
you don't get the wordmark this way).

{{< figure src="/images/sutd-logo-image-search.png" alt="Google Image Search for the SUTD logo" caption="\"sutd logo\" \"sutd logo transparent\" \"sutd logo white\" \"sutd logo high res\"" link="/images/sutd-logo-image-search.png" >}}
