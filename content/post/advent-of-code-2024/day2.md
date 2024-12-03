---
title: Advent of Code 2024
subtitle: Day 2 - Red-Nosed Reports
author: sol
date: 2024-12-03
tags: ["advent-of-code", "nim", advent-of-code-2024]
---

Let's move on with advent of code day 2. A little harder than yesterday, but
still managable.

# Part 1

> The unusual data (your puzzle input) consists of many reports, one report per line. Each report is a list of numbers called levels that are separated by spaces. The engineers are trying to figure out which reports are safe. The Red-Nosed reactor safety systems can only tolerate levels that are either gradually increasing or gradually decreasing. So, a report only counts as safe if both of the following are true:
>
> - The levels are either all increasing or all decreasing.
> - Any two adjacent levels differ by at least one and at most three.
>   Analyze the unusual data from the engineers. How many reports are safe?

Ok this sounds not to complicated. We can just process the input again line by
line. But first let's read in the input and turn it into lines again:

```nim
let filename = "input.txt"
let input = readFile(filename)
```

Now I will implement the constraits that checks each line:

```nim
proc check(line: seq[int]): bool =
  let asc = (line[1] - line[0] > 0)

  for i in 1 ..< line.len:
    let last = line[i-1]
    let current = line[i]
    let diff = abs(last - current)
    if diff == 0 or diff > 3:
      return false
    if asc:
      if last > current:
        return false
    else:
      if last < current:
        return false
  return true
```

This procedure will return true if a line is safe and false otherwise. Let's go
over it. First I see if the line is ascending or descending with the first two
elements. Then I parse always to elements each time. Checking that it and the last
are still asc or desc and do not differ by more than 3. Thats the whole procedure.

Now whats only left is to use it to count how many safe lines there are:

```nim
import std/strutils
import std/sequtils
import std/strformat

var safe = 0
for line in input.splitLines():
  let line_numbers = line.split().map(parseInt)
  if check(line_numbers): safe += 1
```

echo "Safe reports: ",safe

and thus the first part of day 2 is done. Let's move to the second part.

# Part 2

The second part is where it gets tricky.

> Now, the same rules apply as before, except if removing a single level from an unsafe report would make it safe, the report instead counts as safe.

I first tried a somewhat smart solution where it would remove the first erroneous
number and check the line again. This though resulted in some lines falsly being
tagged as false. As sometimes the error is complicated and a specific digit makes
the report pass even if removing another erroneous digit wouldn't make it safe.
Thus is moved to the "crude" version of just checking all possibilities, by creating
the following procedure:

```nim
proc remover(line: seq[int]): seq[seq[int]] =
  var seq_without: seq[seq[int]] = @[]
  seq_without.add(line[1..^1])
  seq_without.add(line[0..^2])
  for i in 1 ..< line.len-1:
    let left = line[0 ..< i]
    let right = line[i+1 .. ^1]
    seq_without.add(left & right)
  return seq_without & line
```

All it does is remove a single digit from line and the put that into a sequence.
Rinse and repeat for all digit in the line and resulting are all possible lines
without an erroneous digit. All that's left to do is to change how we check line
numbers:

```nim
var safe = 0
for line in input.splitLines():
  var line_numbers: seq[int] = line.split().map(parseInt)
  let is_safe: bool = remover(line_numbers).any(check)
  if is_safe: safe += 1
```

For that I just check if any of the lines parse the check and if they do I add
the report as safe. Although not the optimal version it does the job.

And with that the second day of advent of code is concluded. Hope you had a great
read.
