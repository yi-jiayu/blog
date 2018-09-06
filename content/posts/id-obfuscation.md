---
title: "Methods for obfuscating sequential IDs"
subtitle: "Block ciphers and other constructions"
date: 2018-09-06T22:43:27+08:00
---

Auto-incrementing integer IDs are often used as unique row identifiers in many databases, whether specified
explicitly by the user or implicitly by the database engine. These sequential IDs may be a source of information
leakage, allowing third-parties to [infer statistics about](https://en.wikipedia.org/wiki/German_tank_problem)
or simply enumerate your data.

This is not always a bad thing, and may in fact be intuitive in many contexts involving naturally ordered data. For
example, I think it makes sense for my CI builds to be numbered sequentially (after all, they are already tied to
a unique commit identifier).

Nevertheless, sometimes you just do not want to expose sequential IDs (even if only because
larger numbers or short strings look nicer). While not a replacement for proper access control
and verification, one option is to switch to a non-sequential ID generation algorithm such as
[UUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier). Alternatively, and the topic of this post,
sequential IDs can be obfuscated to appear non-sequential.

## Obfuscation methods

### Hashids

One library for doing this which has been in fashion recently is [Hashids](https://hashids.org/), which maps
integers to short strings like `yr8` or `3kTMd`, inspired by the IDs used by YouTube and URL shorteners.

{{< linkpreview title="Hashids - generate short unique ids from integers"
description="Generate short unique ids from integers. Use in url shortening or as unique ids."
url="https://hashids.org/" >}}

Hashids has some nice features such as trying not to generate strings which contain common English curse words, but it

At the same time, Hashids makes no guarantees of security. There exists at least one algorithm to reverse obfuscated
hashids that performs better than brute force, presented in a blog post in which the author claims that anyone
using Hashids should consider it fully-reversible:

{{< linkpreview title="Cryptanalysis of hashids"
description="Carnage's tech talk"
url="http://carnage.github.io/2015/08/cryptanalysis-of-hashids" >}}

Hashids does have some nice features such as trying not to generate strings which contain common English curse
words, and a wide range of implementations across many programming languages. Depending on your threat model,
it could be a convenient way to prettify your integer IDs.

### Block ciphers

Another way to obfuscate IDs is through symmetric encryption. An ID is encrypted to obtain a
corresponding opaque identifier be exposed, and decrypted before use. This can be done using a [block
cipher](https://en.wikipedia.org/wiki/Block_cipher) since IDs usually have a fixed length.

The Perl `Crypt::Skip32` package has existed since 2007 and mentions scrambling database IDs as one of its main
use cases. It adapts the [Skipjack cipher](https://en.wikipedia.org/wiki/Skipjack_(cipher)) to 32-bit block sizes,
and thus works with 32-bit integer database IDs.

{{< linkpreview title="Crypt::Skip32 - 32-bit block cipher based on Skipjack - metacpan.org"
description="32-bit block cipher based on Skipjack"
url="https://metacpan.org/pod/Crypt::Skip32" >}}

Nowadays it's not uncommon to see 64-bit database IDs as well, and common block ciphers with a 64-bit block size like
[triple DES](https://en.wikipedia.org/wiki/Triple_DES) or [Blowfish](https://en.wikipedia.org/wiki/Blowfish_(cipher))
can be directly used to obfuscate these without modification.

Security wise, it seems to me that such an obfuscation method is directly tied to the strength of the encryption
used, and reversing it should entail breaking the underlying cipher. However, besides the key size and algorithm
itself, the security of a block cipher is also a function of the block size.

While small block sizes are more efficient for encrypting similarly sized IDs, they are also more vulnerable to
collision or birthday attacks, as demonstrated against some widely-used block ciphers:

{{< linkpreview title="Sweet32: Birthday attacks on 64-bit block ciphers in TLS and OpenVPN"
description="Sweet32: Birthday attacks on 64-bit block ciphers in TLS and OpenVPN"
url="https://sweet32.info/" >}}

Fortunately though, the attack above takes advantage of specific [block cipher modes of
operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation), specifically how CBC mode collisions
reveal the exclusive-or of two plaintext blocks, and does not apply to obfuscating individual IDs that fit in a
single block, which is effectively ECB mode instead.

### Others

Some other constructions for a reversible ID obfuscation function besides block ciphers:

- [Optimus](https://github.com/jenssegers/optimus), mapping integers to other integers based on Knuth's integer hash:

{{< linkpreview title="Jens Segers - Id transformation with Optimus"
description="This unbelievable small and fast algorithm will generate random-like integers with the ability to convert them back to the original value."
url="https://jenssegers.com/67/id-transformation-with-optimus" >}}

- A custom combination of various methods including xor, bit shuffling and base conversion, described in [this
StackOverflow answer](https://stackoverflow.com/a/8555047).

{{< linkpreview title="algorithm - Obfuscating an ID - Stack Overflow"
description="I'm looking for a way to encrypt/obfuscate an integer ID into another integer."
url="https://stackoverflow.com/a/8555047" >}}

- The [hasty pudding cipher](https://en.wikipedia.org/wiki/Hasty_Pudding_cipher), technically designed as a block
cipher but more of an academic curiosity, which has a customisable block size.

Such bespoke solutions may have the advantage of being a better fit for their specific use cases.

## Presents: gift wrap your IDs

Just for fun, I created my own library for obfuscating integer IDs based on block ciphers.

{{< linkpreview title="yi-jiayu/presents"
description="Like hashids, but based on block ciphers"
url="https://github.com/yi-jiayu/presents" >}}

Similar to Hashids, it converts positive integers to short strings. It first encrypts the integer with a block
cipher, then converts the encrypted integer to a string by changing its base and encoding it into a custom alphabet.

Presents is implemented in [Go](http://golang.org/) and can use any 64-bit block cipher implementing the standard
library [`cipher.Block`](https://golang.org/pkg/crypto/cipher/#Block) interface. The character set for the output
string can be customised, and optionally further shuffled with a specific random seed as well. It converts an integer
like `1213486160` to `90NyXHLckhA`, although it doesn't do anything fancy to prevent curse words from being generated.

The name comes from the [PRESENT block cipher](https://en.wikipedia.org/wiki/PRESENT), which is currently the default
block cipher used by Presents. We had to implement PRESENT in school in our security class and I wrote versions
of it in both [Go](https://github.com/yi-jiayu/PRESENT.go) and [Rust](https://github.com/yi-jiayu/PRESENT.rs),
inspiring me to write this library as well. However, I do want to [change the default block cipher to a
more widely-used one like triple DES or Blowfish](https://github.com/yi-jiayu/presents/issues/1).
