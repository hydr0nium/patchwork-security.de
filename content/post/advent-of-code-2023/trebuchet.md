---
title: Advent of Code 2023
subtitle: Day 1 - Trebuchet?!
author: sol
date: 2024-07-09
tags: ["advent-of-code", "advent-of-code-2023", "rust"]
---

Today we are talking about the first challenge in the advent of code of 2023.
I know I am late but I thought that now that I am learning the rust programming
language the advent of code challenges would be a nice way of improving and
learning it.

# Part 1

The first challenge is called `trebuchet?!` and
the goal is to extract the first and last number from a line, to concatenate
them and convert them to a number. The last step is to add up all the integers.
Here is an example input:

```
1abc2 -> 12
pqr3stu8vwx -> 38
a1b2c3d4e5f -> 15
treb7uchet -> 77

Sum == 142
```

We see that if only one number is present then we will pick that number in both
the first and last number case. Our result is thus `142`. My goal with the solution
I constructed was to understand and practice the concept of working with types that
implement the `Iterator Trait`. First thing was to read in the file with:

```rust
fn read_file() -> String {
    fs::read_to_string("./input.txt")
    .expect("Could not read from input file!")
}
```

To make my life easier I created a a combinator chain that would reduce each line
to only the numbers. For Example:

```
1abc2 -> 12
pqr3stu8vwx -> 38
a1b2c3d4e5f -> 1235
treb7uchet -> 7
```

```rust
let only_ints: Vec<_> = input
        .lines()
        .into_iter()
        .map(|val| val.chars().filter(|c| c.is_numeric()))
        .map(|n| n.collect::<String>())
        .collect();
```

With this vector of 'ints' it was pretty easy to grab the first and last character from each line, convert them into an proper int type and sum them all up:

```rust
let res: u32 = only_ints
        .into_iter()
        .map(|s| {
            s.chars()
                .next()
                .map(|c| {
                    s.chars().next_back().map(|k| {
                        (c.to_string() + &(k.to_string()))
                            .parse::<u32>()
                            .expect("Could not form a int from the two substrings!")
                    })
                })
                .flatten()
        })
        .flatten()
        .sum::<u32>();
```

It might not be the best way to solve it but at least it works. I wonder if there is a better way to grab the first and last element from each line. All thats left was to print `res` and thus we solved the first part of day 1.

# Part 2

For the second challenge the goal was to the basically the same but now also substrings like `one` or `nine` counted as number. The tricky thing was that they sometimes overlapped. For example:

```
nctwonefourjzgskmxjmq2
```

Here `two` and `one` both share the `o` thus a simple replace is not enough because it would destroy the `one` or the `two`. My solution to this was to iterate over the string checking if it begins with any string like `one`, `two`, etc. Then I would convert that to a number, add it to the result output, remove the first char of the string and do the same thing again until I consumed the whole string. If it didnt match something at the start I would still remove the first char, add it to the result and recursively iterate over the rest of the string:

```rust
fn replace_ascci_num(line: &str) -> String {
    if line.len() == 0 {
        return String::from("");
    }
    let nums = [
        "one", "two", "three", "four", "five", "six", "seven", "eight", "nine",
    ];
    for (num, s) in nums.iter().enumerate() {
        if line.starts_with(*s) {
            let r = (num + 1).to_string() + &(replace_ascci_num(&line[1..]));
            return r;
        }
    }
    let r = String::from(&line[0..1]) + &(replace_ascci_num(&line[1..]));
    return r;
}
```

Now all that was left was to map that onto each line before we did the same thing as for [part 1](#part-1):

```rust
let input: String = read_file();
    let results: Vec<_> = input
        .lines()
        .into_iter()
        .map(|line| replace_ascci_num(line))
        .map(|line| line.chars().filter(|c| c.is_numeric()).collect::<String>())
        .flat_map(|line| {
            line.chars().next().map(|f| {
                line.chars().next_back().map(|l| {
                    (f.to_string() + &(l.to_string()))
                        .parse::<u32>()
                        .expect("Could not form a int from the two substrings!")
                })
            })
        })
        .flatten()
        .collect();
    dbg!(results.iter().sum::<u32>());
```

Thats it! This are both solutions to the first day of advent of code 2023 day one. I hope you had fun reading it. For all the code you can check out the git repo at [Github](https://github.com/hydr0nium/advent_of_code_2023/blob/main/trebuchet_day1/src/main.rs).
