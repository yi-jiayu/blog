---
title: "Sudoku addendum"
subtitle: "An unexpectedly problematic input"
date: 2019-03-14T10:19:40+08:00
---

This is a follow-up from my [previous post]({{< relref "/posts/sudoku.md" >}}) about writing a
simple backtracking sudoku solver.

Recently I generated another puzzle on [QQWing](https://qqwing.com/generate.html) which was
considered easy:

```
..6..5....9.1.3..5.....27........9..8...6..7.......1.6..358..2.6....1.5..7...96.1
Number of Givens: 25
Number of Singles: 35
Number of Hidden Singles: 21
Number of Naked Pairs: 0
Number of Hidden Pairs: 0
Number of Pointing Pairs/Triples: 0
Number of Box/Line Intersections: 0
Number of Guesses: 0
Number of Backtracks: 0
Difficulty: Easy
```

And yet my solver takes even longer than it needed on the adversarial input:

```console
$ time echo ..6..5....9.1.3..5.....27........9..8...6..7.......1.6..358..2.6....1.5..7...96.1 | python3 sudoku.py 
146875293297143865358692714465217938831964572729358146913586427684721359572439681

real	1m0.438s
user	1m0.244s
sys	0m0.095s
```

I wonder what about this puzzle makes it difficult for the algorithm?

