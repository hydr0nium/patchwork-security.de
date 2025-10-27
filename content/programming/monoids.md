+++
author = "sol"
title = "The hero of programming: The Monoid"
date = 2024-07-24
description = "When talking about monoids its important to first understand what it means to be a monoid. A monoid, mathematically speaking is a structure with a binary operation, that has a neutral or identity element, is closed, and obeys the law of associativity, but that definition without any context is not easy to understand."

[taxonomies]
tags = ["programming", "monoid", "category-theory"]
+++

# What is a Monoid?

When talking about monoids its important to first understand what it means to be a monoid. A monoid, mathematically speaking is a structure with a binary operation, that has a neutral or identity element, is closed, and obeys the law of associativity, but that definition without any context is not easy to understand. For example:

- What is a binary operation?
- What does closed mean?
- What is a identity / neutral element?
- What is the law of associativity?

To answer all of these questions let's start at the beginning:

## 1. Sets

```
A set is a collection of things.
```

Thats it![^1] Here are some examples of sets:

- The collection of `my university notes` form a set
- The collection of `all natural numbers (0, 1, 2, 3, ...)` is a set written as ℕ
- The collection of the elements of the `String` type in programming
- ...

Basically anything you can think of that can be put into a hypothetical box is a set.[^1] As we also see sets can be infinite like the set of `natural numbers` or the `String` set. Sets without anything are pretty boring. It's like a car without fuel. It exists but it can not go anywhere. So let's look at the "fuel" of sets:

## 2. Binary Relation[^2]

```
A binary relation is a relationship between two elements of a set.
```

Equality (=) is a type of binary relation it takes two elements and produces a truth (boolean) value.

```
5 = 5 -> true
1 = 0 -> false
...
```

## 3. Binary Operation

A binary relation can also 'return' an element from the underlying set. Such a relation is known as a `binary operation`. The operation of addition (+) is such a binary operation:

```
1+1 -> 2
0+7 -> 7
32+32 -> 64
...
```

Notice though that `order` is important. In the case of addition on natural numbers the order does not matter. But considering Strings with concatenation (+) it does:

```
"Hello" + "World" =/= "World" + "Hello"
```

## 4. Closedness

But what does it mean for a binary operation to be closed? Let's consider the operation of `subtration` (-) on the natural numbers (0, 1, 2, 3, ...):

```
5-2 -> 3
1-6 -> -5
```

As we can see the second example poses a problem. When we restrict ourselves only to the natural numbers (non negative whole numbers) and subtract 1 and 6 we don't get a natural number back. This means the operation of subtraction on the natural numbers is not closed.

```
A operation is closed if for any two inputs from the set S the output is also in the set S.
```

## 5. Neutral / Identity Element

The sets we have currently investigated have one special element.[^3] For natural numbers it's the zero element, and for Strings it's the empty string "". The special property of these element is that if they are combined with any other element the element is unchanged:

```
7+0 = 0+7 -> 7
"" + "Hello world!" = "Hello world!" + "" -> "Hello world!"
```

```
If a set with an operation has an element e that we can combine with any other element and it the result is that other element then e is called the neutral or identity element.
```

## 6. Associativity

Associativity is just a fancy term to refer to the fact that the order of operations does not matter:

```
(1+2)+3 -> 6
1+(2+3) -> 6
```

This fact is so blatantly obvious that it sometimes confused people why we even state it explicitly, or why it should be important to note. First consider the operation of subtraction again:

```
(1-2)-3 -> -4
1-(2-3) -> 2
```

As we can see subtraction is not associative. Associativity allows as to just forget the order in which we evaluate an expression with multiple operants and leave out the brackets.

```
(1+2)+3 = 1+2+3
```

## 7. Monoid

Now we know all the components that make up a monoid. A set with an binary operation that:

- is associative
- has a neutral element
- is closed

## 8. Examples of Monoids

We have already seen above some sets that satisfy all the properties stated above, but here are some more:

- Natural Numbers with addition and multiplication
- Strings with concatenation
- Ints[^5] with addition and multiplication
- Lists with concatenation
- ...

## 9. Why do we care about monoids?

You might be wondering why we go all about the definition of monoids. Monoids are maybe special for mathematicians but for programmers they are especially interesting. Why? Because they allow us to do magic:

### 9.1 `The Folder or Reducer`

Let's say we have a list[^4] of elements of a monoid. As an example we will take the set of natural numbers again with the operation of addition. Now associativity allows use a special feature namely that of the reduction of that list:

```py
reduce([1,2,3,4,5,6,7,8,9,10])
```

What the reduction does is to put a binary operation between all the elements and evaluate it as such:

```py
1+2+3+4+5+6+7+8+9+10 -> 55
```

This is only possible because the order of the operation evaluation does not matter. Hence associativity makes it possible to reduce lists of monoid elements.

We can generalize this even more by adding what type of monoid operation we would like to use. This would look like this:

```py
reduce([1,2,3,4,5,6,7,8,9,10], +) # 55
reduce([1,2,3,4,5,6,7,8,9,10], *) # 3628800
```

There is still one more problem:

```
What happens if we reduce the empty list?
```

To solve the problem of reducing a empty list we need to use the last property of monoids. That of a neutral element. If we tell change reduce into the following form:

```py
reduce([1,2,3,4,5,6,7,8,9,10], +, 0) # 55
reduce([], +, 0) # 0
reduce([], +, 1) # 1
```

Now reduce is defined on all possible lists of our monoid. Usually this form of reduce is called `fold`. What else does a monoid allow us to do?

### 9.2 `The Parallelizer`

Associativity does not only solve the problem of condensing lists of monoids to a single element but also allows us to implement trivial parallelism. To do this notice first that we can place the brackets like we want:

```
(1+2)+(3+4)+(5+6)+(7+8)+(9+10)
  |     |     |     |     |
  v     v     v     v     v
  3  +  7  +  11 +  15 +  19
```

Now we can parallelize each of these brackets via a subtask. The well-definedness of this is ensured by the associativity. We can either reduce now or repeat the process until we have a solution. This makes this very fast.

### 9.3 `The MapReducer`

Now that we have a basic understanding of how we can use monoids we can look at one of the best applications for a monoid. That of a `map-reduce`. Map-reduce is a more sophisticated version of fold or reduce. It can be implemented like this:

```py
s = some_list
f = function
b = binary_operation
e = neutral_element

map_reduce(s, f, b, e) = reduce(map(s, f), b, e)
```

But what does the map method do? Map takes some list. This list does not need to be a list of monoid elements. It can be any list of any type. Even list of multiple types can be used like the one in python. The function is special though. The function has the type `s -> m` where it takes any element of the list (s) to a element of the monoid (m). The only requirement is that the monoid stays consistent. Let's look what we could do with this:

```py
filter_strings(s) = if len(s) >= 5: s else ""

map_reduce(["hello","this", "world", "is", "nice"], filter_strings, +, "") # hello world
```

Although this is a very simple example it is a very powerful tool that allows us to do all sorts of things. It basically creates structure out of unstructured data and can be defined instantly on any type to is a monoid. Map-reduce is also know as a `foldMap` in some programming languages such as Haskell. Let's see some application for all the stuff we have learned. One of the best applications is that map_reduce can be used to implement all sorts of functions:

```py
id(x) = x

one(x) = 1

map(x, f) = [f(x)]

filter(x,f) = if f(x): [x] else []

sum(number_list) = map_reduce(number_list, id, +, 0)

product(number_list) = map_reduce(number_list, id, *, 1)

join(string_list) = map_reduce(string_list, id, +, "")

len(some_list) = map_reduce(some_list, one, +, 0)

list_map(some_list, f) = map_reduce(some_list, map(_, f), +, [])

filter_map(some_list, f) = map_reduce(some_list, filter(_, f), +, [])
```

There is much more you could do with foldMap / map-reduce but I think this gives you a good overview on how powerful monoids can be.

### Footnotes:

[^1]: We don't need to worry to much about any advanced definitions for sets here. Naive set theory is enough.
[^2]: A binary relation can be also be defined as a subset (R) of the set of tuples of the underlying set (S). $$R \subseteq S^2$$ We usually write x R y if x and y are in a relationship.
[^3]: The fact that this special element is unique is easy to proof: $$\text{Let } e' \text{ and } e \text{ both be neutral elements with the + operation:} \\ e'+e=e'=e+e'=e \\ \implies e'=e$$
[^4]: We aren't necessarily talking about lists in the programming sense but you can think of them as such. A more abstract way of thinking about them would be a n-tuple or ordered set.
[^5]: Ints can overflow but they still qualify as a monoid.
