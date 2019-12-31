---
title: "Automatically solving picross puzzles from a screenshot"
subtitle: "For when you're wrong on the last cell"
date: 2019-11-01 00:10:27+08:00
tags: ["picross", "python", "z3", "numpy"]
---

I recently worked on a picross solver that solves puzzles from a screenshot of
the puzzle.

The solver uses traditional image processing techniques (without machine
learning) to recognise puzzle hints from a screenshot, and constraint
programming to find a solution.

It is written in Python and relies on the
[scikit-image](https://scikit-image.org/) library for image processing and the
[Z3 Theorem Prover](https://github.com/Z3Prover/z3) for constraint solving.

Though still a work in progress, the source code can be found on GitHub:

{{< linkpreview title="yi-jiayu/picrosser"
description="Z3-powered picross solver"
url="https://github.com/yi-jiayu/picrosser" >}}

## Background

Picross, or nonograms, are a type of puzzle similar to sudoku which look like
this:

{{< figure src="puzzle.png" caption="Screenshot of a picross puzzle from <a href=\"https://play.google.com/store/apps/details?id=com.tuesdayquest.logicart\">Hungry Cat Picross</a> on Android" >}}

The objective is to colour in all the cells on the grid according to the hints
for each row and column (the numbers at the top and left side). Each hint
indicates the number of cells which should be filled with a certain colour in
the row or column it applies to. Circled hints mean that the cells of that
colour should be contiguous.

Here's what the solution to the puzzle above looks like:

{{< figure src="solution.png" caption="The completed puzzle" >}}

## Picross as a constraint satisfaction problem

Similar to a sudoku puzzle, a picross puzzle can be modelled as a constraint
satisfaction problem, with the hints being the constraints on each row and
column.

### Boolean algebra

The following picross puzzle only has one colour. Hints which refer to
consecutive cells are represented by negative numbers.

<table>
  <tr>
    <th></th>
    <th>1</th>
    <th>-2</th>
    <th>2</th>
  </tr>
  <tr>
    <th>2</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>1</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>-2</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

Assuming that each cell is a boolean variable that is true if the cell is
filled, we can come up with a boolean expression which indicates whether a row
or column is coloured correctly.

Let's focus on a single row, with each cell given a label:

<table>
  <tr>
    <th>1</th>
    <td>a</td>
    <td>b</td>
    <td>c</td>
  </tr>
</table>

The hint is 1, so only one of cells A, B and C can be filled, and the other two
will be empty. This can be written in boolean algebra as `(a ∧ ¬b ∧ ¬c) ∨ (¬a ∧
b ∧ ¬c) ∨ (¬a ∧ ¬b ∧ c)`, which means:

```
a AND (NOT b) AND (NOT c)  -  a is coloured, but b and c are not
OR
(NOT a) AND b AND (NOT c)  -  b is coloured, but a and c are not
OR
(NOT a) AND (NOT b) AND c  -  c is coloured, but a and b are not
```

These are the only 3 possibilies for these 3 cells given the hint 1.

We can do the same for another row:

<table>
  <tr>
    <th>-2</th>
    <td>a</td>
    <td>b</td>
    <td>c</td>
  </tr>
</table>

The hint -2 means that there are two coloured cells in this row, but they also
have to be connected. There are only two possibilities this time:

```
a AND b AND (NOT c)  -  a and b are coloured while c is not
OR
(NOT a) AND b AND c  -  b and c are coloured while a is not
```

How about the following row?

<table>
  <tr>
    <th>2</th>
    <td>a</td>
    <td>b</td>
    <td>c</td>
  </tr>
</table>

There's only one way to have two cells filled without them connecting---a and c
have to be filled but not b. However, there will be more possibilities with a
longer row. Rather than enumerate all the possible combinations in boolean
algebra, later we'll take a shortcut and use other types of constraints available in Z3.

Finally, the case for a hint of 0 is trivial.

By applying these constraints to each row and column, we can solve for the
correct colouring of the puzzle:

<table>
  <tr>
    <th></th>
    <th>1</th>
    <th>-2</th>
    <th>2</th>
  </tr>
  <tr>
    <th>2</td>
    <td>x</td>
    <td></td>
    <td>x</td>
  </tr>
  <tr>
    <th>1</td>
    <td></td>
    <td>x</td>
    <td></td>
  </tr>
  <tr>
    <th>-2</td>
    <td></td>
    <td>x</td>
    <td>x</td>
  </tr>
</table>

For a puzzle with multiple colours, each cell will have a boolean variable for
each colour, and a constraint has to be added that each cell can only have one colour.

### Using Z3

We can almost directly translate our problem for Z3 in Python with the `z3-solver` package:

```python
from z3 import And, Bool, Not, Or, Solver, sat

a = Bool('a')
b = Bool('b')
c = Bool('c')

constraints = Or(
    And(a, b, Not(c)),
    And(Not(a), b, c),
)

s = Solver()
s.add(constraints)

if s.check() == sat:
    m = s.model()
    print('a:', m.evaluate(a))
    print('b:', m.evaluate(b))
    print('c:', m.evaluate(c))
else:
    print('unsat')
```

`Bool` creates a boolean variable, while `And`, `Or` and `Not` directly map to
boolean algebra operations.

Running this, we get:

```
a: False
b: True
c: True
```

The solver only returns the first solution it finds. There are ways to find all
possible solutions, however generally there should only be one solution to a
given picross puzzle so that's enough for us.

My actual solver implementation takes a map of puzzle attributes containing the
number of rows, columns and colours in the puzzle and a nested array of hints
for each row and colour, another one for the columns, and the actual colours to
use to generate an image of the solution.

Here's the puzzle from the start of the post represented in
[TOML](https://github.com/toml-lang/toml):

```toml
nrows = 15
ncols = 10
ncolours = 4
rows = [
  [0, 0, -7, -3],
  [0, 0, -4, -6],
  [0, 0, -3, 7],
  [0, 0, -6, 4],
  [0, 0, -4, -6],
  [0, 1, 4, -5],
  [0, -3, -4, -3],
  [0, -4, -3, 3],
  [0, -5, -4, 1],
  [0, -6, -4, 0],
  [0, -7, -3, 0],
  [-2, -8, 0, 0],
  [1, -9, 0, 0],
  [0, -10, 0, 0],
  [-10, 0, 0, 0]
]
columns = [
  [3, 1, 5, 6],
  [2, -2, 6, 5],
  [1, -3, -8, -3],
  [1, -4, 7, 3],
  [1, -5, 4, 5],
  [1, -6, 2, 6],
  [1, -7, -4, -3],
  [1, -8, -4, -2],
  [1, -9, -3, -2],
  [1, -8, 3, -3]
]
colours = [[211, 175, 83], [136, 93, 0], [155, 151, 219], [255, 255, 255]]
```

It outputs a textual representation of the puzzle solution (squint and you can
sort of see it):

```
[4, 4, 4, 3, 3, 3, 3, 3, 3, 3]
[4, 4, 4, 4, 4, 4, 3, 3, 3, 3]
[4, 4, 4, 4, 4, 4, 3, 3, 3, 4]
[4, 4, 3, 3, 3, 3, 3, 3, 4, 4]
[3, 3, 3, 3, 4, 4, 4, 4, 4, 4]
[3, 3, 3, 4, 4, 4, 4, 4, 2, 3]
[3, 3, 3, 3, 4, 4, 4, 2, 2, 2]
[4, 4, 3, 3, 3, 4, 2, 2, 2, 2]
[4, 3, 3, 3, 3, 2, 2, 2, 2, 2]
[3, 3, 3, 3, 2, 2, 2, 2, 2, 2]
[3, 3, 3, 2, 2, 2, 2, 2, 2, 2]
[1, 1, 2, 2, 2, 2, 2, 2, 2, 2]
[1, 2, 2, 2, 2, 2, 2, 2, 2, 2]
[2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
[1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
```

As well as an image:

{{< figure src="output.png" alt="The generated puzzle solution" >}}


Note that this is not the only way to formulate picross as a constraint
satisfaction problem. For example, instea d of using boolean variables and
separate grids for each colour, we could have a single grid and integer
variables to represent colours.

<hr>

In another post, I'll describe how the solver recognises the puzzle hints from
a screenshot.
