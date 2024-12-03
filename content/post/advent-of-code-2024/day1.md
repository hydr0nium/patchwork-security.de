---
title: Advent of Code 2024
subtitle: Day 1 - Historian Hysteria
author: sol
date: 2024-12-03
tags: ["advent-of-code", "nim", advent-of-code-2024]
---

As most years I try to do the advent of code challenges. Sadly I haven't posted
anything except the [first day last year](/post/advent-of-code-2023/trebuchet/). Although this, this years language
will be [nim](https://nim-lang.org/). Nim is a "statically typed compiled
systems programming language". Nim can be thought of as Python but compiled.
There are some quicks here and there but overall the language is fully functional.
Some problems the language currently faces is that there a people who write
malware with it, as it is easy to code with and can cross-compile. Thus Windows
Defender flags the nim compiler and some nim programs as malware. I hope that
this will get better with time as nim is an absolute beast of a language. So
let's begin with day 1 of advent of code: Historian Hysteria.

# Part 1

Here is a quick problem description:

> Pair up the smallest number in the left list with the smallest number in the right list, then the second-smallest left number with the second-smallest right number, and so on. Within each pair, figure out how far apart the two numbers are; you'll need to add up all of those distance.

This is fairly straight forward. We first parse the input into two list, sort them,
calculate the distances, and then sum all of that up. Let's get going:

First we need to parse the input. This is fairly easy and can be done in nim like
this:

```nim
let example_input = readFile("input.txt")
```

Then we need to split the input into lines. For that we need the standard module
`strutils`:

```nim
let example_input = readFile("input.txt")
let lines = example_input.splitLines()
```

Now let's define the two sequences and then parse each line into them.
For this nim provides us with a pretty handy procedure:
[`splitWhitespace()`](https://nim-lang.org/docs/strutils.html#splitWhitespace.i,string,int)
What this procedure does is it splits the string at whitespaces and then strips
leading and trailing whitespaces. This results in perfect output for us to turn
both left and right part into numbers. These just need to be added to the
respective sequences.

```nim
var left: seq[int] = @[]
var right: seq[int] = @[]



for line in lines:
  let splitted = line.splitWhitespace()
  var left_int = parseInt(splitted[0])
  var right_int = parseInt(splitted[1])
  left.add(left_int)
  right.add(right_int)
```

Now we can sort both sequences using the `std/algorithm` module from the
standard libary. All thats left now is to go over the sequences and calculate
their difference and sum all them up:

```nim
left.sort()
right.sort()

assert left.len == right.len


var sum = 0
for n in 0 ..< left.len:
  let diff = abs(left[n] - right[n])
  sum = sum + diff

echo "Sum is: ", sum
```

The assert is just added to make sure I can use either list to iterate. Now
let's go to the second part of day 1.

# Part 2

> This time, you'll need to figure out exactly how often each number from the left list appears in the right list. Calculate a total similarity score by adding up each number in the left list after multiplying it by the number of times that number appears in the right list.

That shouldn't be much of a hassle. We can just change our original program a bit.
Instead of using,

```nim
let diff = abs(left[n] - right[n])
sum = sum + diff
```

we can use,

```nim
let count = count(right, left[n])
sum = sum + (left[n]*count)
```

from the `std/sequtils` module.

This will count how often the number on the left at index n appears in the
right sequences. Num we only need to multiply according to the problem
description and finished is the second part:

```nim
import strutils
import std/algorithm
import std/sequtils

let example_input = readFile("input.txt")
let lines = example_input.splitLines()
var left: seq[int] = @[]
var right: seq[int] = @[]



for line in lines:
  let splitted = line.splitWhitespace()
  var left_int = parseInt(splitted[0])
  var right_int = parseInt(splitted[1])
  left.add(left_int)
  right.add(right_int)

left.sort()
right.sort()

assert left.len == right.len


var sum = 0
for n in 0 ..< left.len:
  let count = count(right, left[n])
  sum = sum + (left[n]*count)

echo "Sum is: ", sum
```

And that is how you solve day 1 of advent of code using nim. Probably not the
best solution, but hey I am still learning. Happy first advent.
