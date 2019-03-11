---
title: "Solving sudoku"
subtitle: "A simple backtracking sudoku solver"
date: 2019-03-11 22:40:21+08:00
---

As much as I would like to believe that programming has developed my analytical thinking, I am quite
terrible at solving sudoku puzzles, so I decided try writing a sudoku solver instead.

Here is a simple backtracking solver written in Python 3:

```python
def rows(puzzle):
    return [puzzle[9 * i:9 * i + 9] for i in range(9)]


def columns(puzzle):
    return [puzzle[i::9] for i in range(9)]


def boxes(puzzle):
    return [puzzle[i:i + 3] +
            puzzle[i + 9:i + 9 + 3] +
            puzzle[i + 9 + 9:i + 9 + 9 + 3] for i in
            (j * 3 + k * 27 for j in range(3) for k in range(3))]


def valid_group(group):
    digits = [d for d in group if d != '.']
    return len(set(digits)) == len(digits)


def valid(puzzle):
    return all(valid_group(group) for group in rows(puzzle)) and all(
        valid_group(group) for group in columns(puzzle)) and all(
        valid_group(group) for group in boxes(puzzle))


def solved(puzzle):
    return valid(puzzle) and '.' not in puzzle


def permutations(puzzle):
    return [puzzle.replace('.', f'{i}', 1) for i in range(1, 10)]


def solve(puzzle):
    stack = [puzzle]
    while stack:
        curr = stack.pop()
        if solved(curr):
            return curr
        stack.extend(perm for perm in permutations(curr) if valid(perm))


def valid_input(inp):
    return len(inp) == 81 and not set(inp) - set('.123456789')


def main():
    puzzle = input()
    if not valid_input(puzzle):
        print('Invalid input!')
        return
    solved_puzzle = solve(puzzle)
    if not solved_puzzle:
        print('No solution!')
        return
    print(solved_puzzle)


if __name__ == '__main__':
    main()
```

It's also available as a GitHub gist at
https://gist.github.com/yi-jiayu/ef2d83e26db5fd22fb3a86356df8076d.

## Usage

The solver takes a sudoku puzzle written in one line with blanks denoted by periods like this from
standard input:

```
....5.264..7..9..35..62....8......9.7..34..86.....2.3....9.....9.476........3....
```

And writes the solution in a similar format:

```console
$ echo ....5.264..7..9..35..62....8......9.7..34..86.....2.3....9.....9.476........3.... | python3 sudoku.py 
389157264267489153541623978813576492792341586456892731635914827924768315178235649
```

For testing, I came across this website which can generate sudoku puzzles in the appropriate format:

{{< linkpreview title="QQWing - Generate Sudoku Puzzles" description="Generate Sudoku in your web browser using QQWing." url="https://qqwing.com/generate.html" >}}

It can also solve them too, which helped me check the correctness of the solver's output.

## Handling adversarial input

The Wikipedia page [Sudoku solving
algorithms](https://en.wikipedia.org/wiki/Sudoku_solving_algorithms#cite_note-difficult_17_clue-1)
has a section on the backtracking method used by this solver, and mentions that inputs that are
likely to take significantly longer for such a solver compared to random can be constructed.

The [linked reference](https://www.flickr.com/photos/npcomplete/2361922699), showing a visualisation
of a backtracking solver working on such an adversarial input, also includes a sample which the
author described as taking 6 hours to run on his machine in 2008:

```
---------
-----3-85
--1-2----
---5-7---
--4---1--
-9-------
5------73
--2-1----
----4---9
```

Converting it to the right format and trying it with my solver on my 2018 MacBook Pro, it clearly
does take longer than the first input shown earlier:

```console
$ time echo ....5.264..7..9..35..62....8......9.7..34..86.....2.3....9.....9.476........3.... | python3 sudoku.py
389157264267489153541623978813576492792341586456892731635914827924768315178235649

real	0m0.717s
user	0m0.697s
sys	0m0.016s
$ time echo ..............3.85..1.2.......5.7.....4...1...9.......5......73..2.1........4...9 | python3 sudoku.py
987654321246173985351928746128537694634892157795461832519286473472319568863745219

real	0m6.235s
user	0m6.203s
sys	0m0.019s
```

However, technology has come a long way in more than 10 years---the code is unlikely to win any
performance awards, but instead of taking 6 hours, it takes just over 6 seconds on my machine today!

