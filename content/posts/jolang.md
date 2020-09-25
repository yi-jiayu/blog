---
title: "Jolang"
subtitle: "I wrote my first compiler (sort of)!"
date: 2020-09-25T20:02:03+08:00
---

Jo is a Lisp dialect that compiles directly to Go source code.

{{< linkpreview title="yi-jiayu/jolang"
description="A Lisp dialect which compiles directly to Go source code"
url="https://github.com/yi-jiayu/jolang" >}}

This is a [fizzbuzz](https://wiki.c2.com/?FizzBuzzTest) solution in Jo:

```
(package main)

(import "fmt")

(func main ()
    (for (define i 0) (< i 100) (inc i)
        (if (= 0 (% i 15))
            (fmt.Println "fizzbuzz")
            (if (= 0 (% i 3))
                (fmt.Println "fizz")
                (if (= 0 (% i 5))
                    (fmt.Println "buzz")
                    (fmt.Println i))))))
```

And the corresponding Go code ([playground
link](https://play.golang.org/p/DEssauIdJcq)):

```go
package main

import "fmt"

func main() {
	for i := 0; i < 100; i++ {
		if 0 == i%15 {
			fmt.Println("fizzbuzz")
		} else {
			if 0 == i%3 {
				fmt.Println("fizz")
			} else {
				if 0 == i%5 {
					fmt.Println("buzz")
				} else {
					fmt.Println(i)
				}
			}
		}
	}
}
```

## Building Jo

The Jo compiler is really just a parser that takes Jo source code as input and
produces a Go AST, which can be converted to Go source code using the standard
library's [go/printer](https://golang.org/pkg/go/printer/) or
[go/format](https://golang.org/pkg/go/format/) packages so nothing more has to
be done.

I implemented the parser using [parser
combinators](https://en.wikipedia.org/wiki/Parser_combinator), starting by
adapting the article [Learning Parser Combinators With
Rust](https://bodil.lol/parser-combinators/) into Go.

After moving on to parsing Jo code, I had an "Aha!" moment when I realised how
all the parsers I was writing could be combined into a single parser which
directly returned the entire AST. Before that, I had actually mixed up lexical
analysis and parsing, due to a preconception that compilers always had separate
lexical analysis and parsing steps.

From that point on, it was a relatively mechanical task of referring to the [Go
Programming Language Specification](https://golang.org/ref/spec) and building
parsers for each syntax element, then checking the output against actual ASTs
parsed from Go programs. The tool
[astextract](https://github.com/lu4p/astextract) was invaluable while doing
this.

The biggest challenge was determining how a given Go language construct should
be written in Jo. Certain decisions were easy to make, like writing `package
main` as `(package main)` or using `(f a b c)` for function call syntax (because
Lisp). But decisions like using `(define x 0)` for short variable
declaration `x := 0` and `(assign x 0)` for normal assignment `x = 0` were
mostly stylistic, and decisions like sometimes requiring an additional set of
parentheses around expression lists were just to make parsing easier.

## Wishlist

From the start Jo was just a learning project, there's absolutely no reason to
use it other than for fun, unlike some other hosted Lisps like
[Clojure](https://clojure.org/index) on the JVM or
[Hy](https://docs.hylang.org/en/stable/) in Python.

Nevertheless, I still want to continue improving it, and apart from supporting
more Go features (key features like slices, struct literals and goroutines are all not
implemented yet), there's one thing I really want to add: macros!

What is a Lisp without metaprogramming? At one point I tried to add a feature to
Jo that wasn't present in Go in the form of an `assert` statement, but in the
process of doing so, I realised that rather baking it into the compiler, it was
just one of many possible language extensions with a macro system.

In fact, code generation via external tools is something that Go has [clearly
embraced](https://blog.golang.org/generate), and Jo itself *is* a Go
code-generating tool, so I think adding a macro system makes sense.

At least that's the plan---I'm not an experienced Lisp programmer nor have I
done much metaprogramming.

Another feature I've thought about is type inference. Go already has some form
of type inference in short variable declarations, and it follows that Jo does as
well. But if Jo had a more advanced type system, it could become more than the
Go AST mapped to S-expressions. For example, maybe function literals could be
written in a more compact form but compile to appropriately-typed Go function
literals.

I've got no idea how it would turn out though. Such a feature would definitely
increase Jo's complexity exponentially since it would no longer be just a dumb
syntax transformation.

Finally, I'm also interested in further exploring parsing techniques. Apparently
parser combinators are a form of [recursive descent
parser](https://en.wikipedia.org/wiki/Recursive_descent_parser) which is one of
the more basic strategies, for reasons related to terms I don't understand right
now like "LL", "LR" and "left recursion". Specifically, I'd like to see how I
could use [ANTLR](https://www.antlr.org/), which is something called a [parser
generator](https://en.wikipedia.org/wiki/Compiler-compiler), to implement Jo
instead.

