---
title: "Life and death in black and white"
subtitle: "A primer on elementary cellular automata"
date: 2018-08-21T18:19:27+08:00
---

{{< aside >}}
After my previous post on PBM and PNG images and ImageMagick, I was interested in doing some more image manipulation,
and decided to explore ways to create PNG files by generating visualisations of elementary cellular automata. This
post provides some background information about cellular automata before that.
{{< /aside >}}

A cellular automaton is a is a pattern of cells on a grid, in which the state of each cell changes over multiple
time steps based on certain rules.

## Life
For example, [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) is a cellular automaton
which consists of cells on a two-dimensional square grid. The state of each cell is either live or dead, and is
updated every time step based on its own state and the states of the 8 cells adjacent to it:

- if a live cell has less than 2 live neighbours, it dies
- if a live cell has exactly 2 or 3 live neighbours, it remains alive
- if a live cell has more than 3 live neighbours, it dies
- if a dead cell has exactly 3 live neighbours, it becomes a live cell

Starting from an initial pattern of cells, the state of the grid in each subsequent time step can be continuously
generated to visualise the evolution of the pattern over time:

{{< figure src="puffertr.gif" >}}
{{% caption  %}}
Conway's Game of Life animation from [Wolfram MathWorld](http://mathworld.wolfram.com/GameofLife.html)
{{% /caption %}}

{{< figure src="Gospers_glider_gun.gif" >}}
{{% caption  %}}
Conway's Game of Life animation from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Gospers_glider_gun.gif)
{{% /caption %}}

## Elementary cellular automata
An elementary cellular automaton ([Wikipedia](https://en.wikipedia.org/wiki/Elementary_cellular_automaton),
[Wolfram MathWorld](http://mathworld.wolfram.com/ElementaryCellularAutomaton.html)) is another specific type of
cellular automaton which exists on a one-dimensional grid (a single row of cells) instead. The state of each cell
is also either live or dead, or on and off, and updated every time step based its own state and the states of its
two neighbours.

### Rules
{{< figure src="01_01_108_60.gif" >}}
{{% caption  %}}
State transition table for a rule-60 cellular automaton from the [Wolfram Atlas](http://atlas.wolfram.com/01/01/60/)
{{% /caption %}}

The table above shows how the state of a single cell can be updated in each time step. The first row contains the
8 possible combinations of its own and its two neighbours' states, and the second row describes what the resulting
state for the middle cell should be in the next time step.

This set of state transitions defines an elementary cellular automaton and is known as its **rule**. The 8 possible
configurations of preceding states and two possible resulting states each give a total of 2<sup>8</sup> or 256
possible combinations of state transitions and hence rules for elementary cellular automata.

The numbering system for rules is known as the [Wolfram code](https://en.wikipedia.org/wiki/Wolfram_code) and
comes from the representation of the state transitions in binary:

{{% overflow-x %}}
| Previous state | 110 | 110 | 101 | 100 | 011 | 010 | 001 | 000 |
|----------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Next state     |  0  |  0  |  1  |  1  |  1  |  1  |  0  |  0  |
{{% /overflow-x %}}

If we interpret the 8 resulting states as a binary number, we get 00111100---or 60 in decimal---hence the labelling
of this rule as rule 60.

### Visualisation

Since a elementary cellular automaton is one-dimensional, a two-dimensional image is enough to visualise its
evolution over time. This is done by concatenating multiple rows of cells representing consecutive time steps
along a second axis.

This is a visualisation of a 10-cell-wide rule-60 cellular automaton with random initial state:

{{< figure src="rule60-t01.png" caption="Initial state" >}}
{{< figure src="rule60-t02.png" caption="Two time steps" >}}
{{< figure src="rule60-t03.png" caption="Three time steps" >}}
{{< figure src="rule60-t10.png" caption="10 time steps" >}}
{{< figure src="rule60.gif" caption="Animation" >}}

Some larger visualisations:

{{< figure src="rule73-2x.png" caption="A rule-73 cellular automaton" >}}

{{< figure src="rule90-2x.png" caption="Sierpinski triangles created by a rule-90 cellular automaton" >}}

{{< figure src="rule110-2x.png" caption="A rule-110 cellular automaton. Rule 110 has been shown to be Turing-complete." >}}

### Edge behaviour
What about the cells at the two ends which only have one neighbour? I'm not sure if there's a standard way to
handle this, but there are 4 main options to choose from:

- Treat the cells beside each end as always off (this is the behaviour I used for the visualisations above).
- Treat the cells beside each end as always on.
- Treat the cells beside each end as having the same state as the cell next to them (reflection). For example,
if the leftmost cell is on, treat its left neighbour as on as well.
- Treat the cells beside each end as having the same state as the cell on the opposite end (wrapping)---this is
like having your simulation take place on a cylinder and the cells on each end are actually touching each other.

(Alternatively, you could also arbitrarily fix or randomise their states if you so wanted.)

### Simulating elementary cellular automata
Here's a small function in Python which will generate the next state for a elementary cellular automaton in the
form of a string of `1`s and `0`s:

```python
# lookup table
patterns = ('111', '110', '101', '100', '011', '010', '001', '000')

def next_state(rule, state, edge_behavior='off'):
    # convert the rule number to 8 binary digits, eg. 60 -> 00111100
    table = format(rule, '08b')

    # create an intermediate state with two "virtual" cells at both ends
    if edge_behavior == 'off':
        intermediate = '0' + state + '0'
    elif edge_behavior == 'on':
        intermediate = '1' + state + '1'
    elif edge_behavior == 'mirror':
        intermediate = state[0] + state + state[-1]
    elif edge_behavior == 'wrap':
        intermediate = state[-1] + state + state[0]
    else:
        raise TypeError('invalid value for edge_behavior')

    # lookup the next state for each cell using a sliding window of size 3 over the intermediate state
    # and join everything together
    return ''.join(table[patterns.index(intermediate[i:i + 3])] for i in range(0, len(intermediate) - 2))
```

Sample usage:

```python
In [2]: r = 60
   ...: s = '1000000000'
   ...: print(s)
   ...: for _ in range(9):
   ...:     s = next_state(r, s)
   ...:     print(s)
   ...:
1000000000
1100000000
1010000000
1111000000
1000100000
1100110000
1010101000
1111111100
1000000010
1100000011
```

On a proper grid, the output would look like this:

{{< figure src="rule60-10x.png" alt="10 time steps of a rule-60 cellular automaton as generated in the code above." >}}

## Conclusion

Elementary cellular automata follow simple rules but can develop complex and interesting
patterns. Not just cosmetic, elementary cellular automata can model real world systems such as
[majority voting](https://en.wikipedia.org/wiki/Majority_problem_(cellular_automaton)), [traffic
flow](https://en.wikipedia.org/wiki/Microscopic_traffic_flow_model#Cellular_automaton_models) and allegedly
[random number generation](https://en.wikipedia.org/wiki/Rule_30#Random_number_generation).

Keep in mind that elementary cellular automata are just a specific type of one-dimensional cellular
automata. You've possibly interacted with cellular automata in many other places before: [falling sand
games](https://en.wikipedia.org/wiki/Falling-sand_game), [fluid simulation](https://sanojian.github.io/cellauto/)
such as in Minecraft, Terraria and Dwarf Fortress, [creature simulation](https://rileyjshaw.com/terra/#creatures),
[cave](https://gamedevelopment.tutsplus.com/tutorials/generate-random-cave-levels-using-cellular-automata--gamedev-9664)
[generation](https://blog.jrheard.com/procedural-dungeon-generation-cellular-automata) and more, all can be done
with cellular automata.
