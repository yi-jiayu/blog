---
title: "From Zero to Zebra in Prolog"
subtitle: "Learning a logic programming language to solve a logic puzzle"
date: 2020-01-18T13:01:16+08:00
tags: ["logic", "prolog"]
---

[Prolog](https://en.wikipedia.org/wiki/Prolog) is a logic programming language,
which involves thinking about problems slightly differently from other
programming languages. I recently picked it up to try solving a logic puzzle
called the [Zebra puzzle](https://en.wikipedia.org/wiki/Zebra_Puzzle).

## First steps

There are multiple implementations of Prolog. Two common implementations are
[GNU Prolog](http://www.gprolog.org/) and
[SWI-Prolog](https://www.swi-prolog.org/). Because most Prolog implementations
follow ISO Prolog, you can get started with any one.

There are three main types of statements in Prolog: facts, rules and queries.
When you first start Prolog (using either `gprolog` for GNU Prolog or `swipl`
for SWI-Prolog), you'll find yourself in an interactive REPL. This is where you
can enter queries.

However, facts and rules are usually defined in a separate file and are loaded
into the REPL by consulting said file. A file can be loaded into the REPL by
typing the filename without the `.pl` extension surrounded by square brackets,
followed by the statement terminator (a full stop). To load a file called
`zebra.pl`, run `[zebra].`.

Try out the simple examples on this site for a quick tour of what you can do
with Prolog:

{{< linkpreview title="Learn Prolog Now!" description="1.1 Some Simple Examples"
url="http://www.learnprolognow.org/lpnpage.php?pagetype=html&pageid=lpn-htmlse1" >}}

To try them from within the REPL, consult the pseudo file `user` to enter them,
then exit with <kbd>Control</kbd>+<kbd>D</kbd> when you're done. For example, in
SWI-Prolog:

```prolog
$ swipl
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.3)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

?- [user].
|:    woman(mia).
|:    woman(jody).
|:    woman(yolanda).
|:    playsAirGuitar(jody).
|:    party.
|: ^D% user://1 compiled 0.00 sec, 5 clauses
true.

?- woman(mia).
true.
```

I mostly based this section on the SWI-Prolog quickstart guide, which was very
helpful for me when I was starting out:

{{< linkpreview title="SWI-Prolog -- Manual" description="2.1 Getting started quickly"
url="https://www.swi-prolog.org/pldoc/man?section=quickstart" >}}

## The Zebra Puzzle

Wikipedia has an article on the zebra puzzle:

{{< linkpreview title="Zebra Puzzle - Wikipedia"
description="The zebra puzzle is a well-known logic puzzle."
url="https://en.wikipedia.org/wiki/Zebra_Puzzle" >}}

The example on the page goes:

> 1. There are five houses.
> 2. The Englishman lives in the red house.
> 3. The Spaniard owns the dog.
> 4. Coffee is drunk in the green house.
> 5. The Ukrainian drinks tea.
> 6. The green house is immediately to the right of the ivory house.
> 7. The Old Gold smoker owns snails.
> 8. Kools are smoked in the yellow house.
> 9. Milk is drunk in the middle house.
> 10. The Norwegian lives in the first house.
> 11. The man who smokes Chesterfields lives in the house next to the man with the fox.
> 12. Kools are smoked in the house next to the house where the horse is kept.
> 13. The Lucky Strike smoker drinks orange juice.
> 14. The Japanese smokes Parliaments.
> 15. The Norwegian lives next to the blue house.
> 16. Now, who drinks water? Who owns the zebra?
>
> Now, who drinks water? Who owns the zebra?
>
> In the interest of clarity, it must be added that each of the five houses is
> painted a different color, and their inhabitants are of different national
> extractions, own different pets, drink different beverages and smoke different
> brands of American cigarets [sic]. One other thing: in statement 6, right
> means your right.

## Solving the Zebra Puzzle

The zebra puzzle is a constraint satisfaction problem. You have to assign one
and only one of each color, inhabitant, pet, beverage and cigarette brand to
each house given the specified conditions.

Apparently it's possible to solve it with basic Prolog, but I kind of cheated
and used a constraint programming library called CLP(FD)---Constraint Logic
Programming over Finite Domains.

In my defense, the [SWI-Prolog documentation on
it](https://www.swi-prolog.org/man/clpfd.html) wrote:

> Almost all Prolog programs also reason about integers. Therefore, it is highly
> advisable that you make CLP(FD) constraints available in all your programs.

Here's my solution in SWI-Prolog:

```prolog
:- use_module(library(clpfd)).

zebra(Colors, Nationalities, Pets, Beverages, Cigarettes) :-
    Colors = [Red, Green, Ivory, Yellow, Blue],
    Nationalities = [English, Spanish, Ukrainian, Norweigian, Japanese],
    Pets = [Dog, Snails, Fox, Horse, Zebra],
    Beverages = [Coffee, Tea, Milk, Orange, Water],
    Cigarettes = [Old, Kools, Chesterfields, Lucky, Parliaments],
    Colors ins 1..5,
    Nationalities ins 1..5,
    Pets ins 1..5,
    Beverages ins 1..5,
    Cigarettes ins 1..5,
    all_different(Colors),
    all_different(Nationalities),
    all_different(Pets),
    all_different(Beverages),
    all_different(Cigarettes),
    English #= Red,
    Spanish #= Dog,
    Coffee #= Green,
    Ukrainian #= Tea,
    Ivory + 1 #= Green,
    Old #= Snails,
    Kools #= Yellow,
    Milk #= 3,
    Norweigian #= 1,
    Chesterfields #= Fox - 1 #\/ Chesterfields #= Fox + 1,
    Kools #= Horse -1 #\/ Kools #= Horse + 1,
    Lucky #= Orange,
    Japanese #= Parliaments,
    Norweigian #= Blue - 1 #\/ Norweigian #= Blue + 1.

% zebra(Colors, Nationalities, Pets, Beverages, Cigarettes), label(Pets), label(Beverages).
```

We start by loading the `clpfd` library to use its constraints, then we declare
a single rule.

The first half of the rule's clauses are for setting up the problem---declaring
all the decision variables and their domains and constraining them to all be
different.

The second half are just rewrites of the 15 statements of the puzzle as CLP(FD)
constraints. Of note are the `#=` and `#\/` operators for equality and logical
disjunction (OR) respectively.

After loading the program, we can pose a query to Prolog to find a satisfiable
set of assignments:

```prolog
$ swipl
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.3)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

?- [zebra].
Warning: zebra.pl:3:
	Singleton variables: [Zebra,Water]
true.

?- zebra(Colors, Nationalities, Pets, Beverages, Cigarettes), label(Pets), label(Beverages).
Colors = [3, 5, 4, 1, 2],
Nationalities = [3, 4, 2, 1, 5],
Pets = [4, 3, 1, 2, 5],
Beverages = [5, 2, 3, 4, 1],
Cigarettes = [3, 1, 2, 4, 5] ;
false.

?-
```

There's a warning about singleton variables because we declared them but did not
use them, but it doesn't affect the solution.

Typing a semicolon tells Prolog to try finding other solutions, but there are no
others so it returns false.

## Differences Between SWI-Prolog and GNU Prolog

While writing this post I was wondering how significant my choice of Prolog
implementation was, so I tried to see if my program would work on GNU Prolog
as well.

Unfortunately, it didn't---it seems like there are differences in the way
CLP(FD) is used in GNU Prolog. Besides having it loaded by default instead of
having to load a library, many of the CLP(FD) predicates have different names.

I managed to modify my program to work in GNU Prolog with some help from the
[documentation](http://www.gprolog.org/manual/html_node/gprolog054.html):

```prolog
zebra(Colors, Nationalities, Pets, Beverages, Cigarettes) :-
    Colors = [Red, Green, Ivory, Yellow, Blue],
    Nationalities = [English, Spanish, Ukrainian, Norweigian, Japanese],
    Pets = [Dog, Snails, Fox, Horse, Zebra],
    Beverages = [Coffee, Tea, Milk, Orange, Water],
    Cigarettes = [Old, Kools, Chesterfields, Lucky, Parliaments],
    fd_domain(Colors, 1, 5),
    fd_domain(Nationalities, 1, 5),
    fd_domain(Pets, 1, 5),
    fd_domain(Beverages, 1, 5),
    fd_domain(Cigarettes, 1, 5),
    fd_all_different(Colors),
    fd_all_different(Nationalities),
    fd_all_different(Pets),
    fd_all_different(Beverages),
    fd_all_different(Cigarettes),
    English #= Red,
    Spanish #= Dog,
    Coffee #= Green,
    Ukrainian #= Tea,
    Ivory + 1 #= Green,
    Old #= Snails,
    Kools #= Yellow,
    Milk #= 3,
    Norweigian #= 1,
    Chesterfields #= Fox - 1 #\/ Chesterfields #= Fox + 1,
    Kools #= Horse -1 #\/ Kools #= Horse + 1,
    Lucky #= Orange,
    Japanese #= Parliaments,
    Norweigian #= Blue - 1 #\/ Norweigian #= Blue + 1.

% zebra(Colors, Nationalities, Pets, Beverages, Cigarettes), fd_labeling(Pets), fd_labeling(Beverages).
```

The predicates for specifying the domain of a variable, adding unique
constraints on variables and labelling were different in GNU
Prolog---`fd_domain` instead of `ins`, `fd_all_different` instead of just
`all_different` and `fd_labelling` instead of `label`.

After modification, the program produces the same solution:

```prolog
$ gprolog
GNU Prolog 1.4.5 (64 bits)
Compiled Aug 20 2018, 15:27:00 with clang
By Daniel Diaz
Copyright (C) 1999-2018 Daniel Diaz
| ?- [gzebra].
compiling gzebra.pl for byte code...
gzebra.pl:1-30: warning: singleton variables [Zebra,Water] for zebra/5
gzebra.pl compiled, 32 lines read - 4683 bytes written, 6 ms

(1 ms) yes
| ?- zebra(Colors, Nationalities, Pets, Beverages, Cigarettes), fd_labeling(Pets), fd_labeling(Beverages).

Beverages = [5,2,3,4,1]
Cigarettes = [3,1,2,4,5]
Colors = [3,5,4,1,2]
Nationalities = [3,4,2,1,5]
Pets = [4,3,1,2,5] ? ;

(1 ms) no
| ?-
```

