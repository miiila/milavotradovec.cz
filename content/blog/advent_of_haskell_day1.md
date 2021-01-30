+++
title = "Advent of Haskell - first few days"
slug = "advent_of_haskell_baby_steps"
date = 2021-01-30
+++
In the [previous article](https://milavotradovec.cz/blog/advent-of-haskell/), I described my Haskell journey and my general feelings. In the upcoming short series of articles, I would like to describe selected problems in more depth. Partially because I want to give you some background about my approach to tasks and partially to give you more info about learning Haskell and functional techniques over selected problems.

Before diving into the first couple of problems, here are rules I set for myself:
- use only the standard library (therefore I know nothing about package management, stack or cabal)
- no regex parsing
- no copy/paste solutions
- keep the pace, start every puzzle on the day when it's released
- have somebody to compete with

All these constraints help me to learn. I need to stay low and get hands-on experience with the basics of a new language. Otherwise, my knowledge gets very shallow and limited. 3rd party libraries and no regex sounds harsh, but I wanted to learn writing Haskell, not moving around Haskell's ecosystem. The last bullet isn't a rule per se, but it helps to stay motivated. I got lucky and had a group of five friends who participated as well. And it was super fun to chat about our different approaches and motivate each other.

For writing and running my code, I used my current go-to editor - vim (specifically [Neovim](https://neovim.io/)) with [LSP](https://github.com/neovim/nvim-lspconfig) and [Haskell Language Server](https://github.com/haskell/haskell-language-server#using-haskell-language-server-with-vim-or-neovim) installed through [ghcup](https://www.haskell.org/ghcup/).


I always wrote a solution first and refactored it later. I'd call it a 2 step learning experience, where you try to come up with the solution as fast as possible (although not competing) and then think how to adapt it. I was also lucky to have a good friend, who has much more experience with Haskell and who was able to patiently review my code every day providing tons of useful feedback. All the solutions (after refactoring) are on [my GitHub](https://github.com/miiila/aoc2020). I will use code samples to illustrate my learning progress. Last comment before diving into code - every puzzle has two parts. However, if you are not logged in, you will see only Part 1 described. I will try to describe part two as briefly as possible.

# [Day 1](https://adventofcode.com/2020/day/1)
In the beginning, the difficulty is usually rather low. Finding two (or three in Part 2) numbers that sum up to a given number was quite easy. The simplest way is to use list comprehension. If you have any Python background, it should sound familiar.
```python
# Python
[x*y for x in input for y in input if x+y == 2020][0] 
```
```haskell
-- Haskell
head [x*y | x <- input, y <- input, x+y == 2020]
```

_[Solution](https://github.com/miiila/aoc2020/blob/main/day1.hs)_
# [Day 2](https://adventofcode.com/2020/day/2)
Second day brought function creation and revealed the usefulness of pattern matching for input parsing. Let's have a look on rather cryptical piece of code:
```haskell
   let (min, _:rest) = break (=='-') s
       (max, _:c:_:_:r) = break (==' ') rest
```
It splits `1-4 j: jjjqzmgbjwpj` into variables `min, max, c, r`. The first expression puts the first number (before `-`) into `min`, skips `-` character and stores `rest`. The second one breaks on first space and parses out `max`, `c` (character before `:`) and `r` (gibberish string). This is very useful for quick and dirty parsing interesting parts of the input and ignoring the rest. `_` stands for nothing, i.e. don't store it anywhere.

One of my favourite building blocks also appears here - `filter`. It works the same as in any other language (e.g. [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)) - it applies a provided function to every list item yielding a new list with those elements, for which function returned true.

Part 2 just changed the meaning of the first two numbers from interval to indexes.

_[Solution](https://github.com/miiila/aoc2020/blob/main/day2.hs)_
# [Day 3](https://adventofcode.com/2020/day/3)
First grid problem! I always struggle with those. Fortunately, this one wasn't that complicated and Haskell helps me here greatly.

The grid is infinitely wide. In most of the languages, you would store the grid in some form of 2d array and you should check the boundaries and extend it if needed (or you would probably copy the array nth times calculating n as a ratio between width and height). Haskell has a much nicer way.

```haskell
map (concat . repeat) (lines contents)
```

`lines` splits the input into lines, `repeat` copies given input infinite times (so you have an infinite list) and `concat` glues the infinite list into an infinite string. Because Haskell evaluates everything lazily, you don't have any issue with having an infinite list. Once you ask for the next item, you will get it.

This day also allowed me to learn more about [`<*> - Applicative Functors`](http://learnyouahaskell.com/functors-applicative-functors-and-monoids#applicative-functors). It is especially handy in Part 2, which requires you to apply an algorithm multiple times using different navigation instructions. Applicative functors allowed me to express the most important part (grid navigation) as functions and feed them the wrapping logic. 

Magical part is here:
```haskell
let slopes = [((+1), (+1)), ((+1), (+3)), ((+1), (+5)), ((+1), (+7)), ((+2), (+1))]
```
It stores a list of tuples in which every item is a function (`(+1)` is just a function expecting one parameter) in a variable. 
```haskell
appliedSlopes = countTrees 0 0 <$> slopes
```
Then a function expecting this tuple as a parameter is mapped over `slopes` variable resulting in a list of new functions (because of [currying](http://learnyouahaskell.com/higher-order-functions#curried-functions)).
```haskell
res = product $ appliedSlopes <*> [input] <*> [0]
```
This list of function is finally applied over an input and "result" (0 at the beginning, used in recursive calls) and multiplied together. Despite not being the best usage here (I had to wrap input and 0 into lists to make it working correctly), it helped me to understand how Applicative Functors work and it paid off quite well later.

_[Solution](https://github.com/miiila/aoc2020/blob/main/day3.hs)_
# [Day 4](https://adventofcode.com/2020/day/4)
Last day of today's writeup and first tedious puzzle that took me much longer than needed. You can see [Map](https://hackage.haskell.org/package/containers-0.4.0.0/docs/Data-Map.html) for the first time (and definitely wrongly) and I also encounter classical AoC troll - while Part 1 is straightforward and nice, Part 2 is completely different and forces you to throw away everything you have and start from scratch. Here, Part 2 brought specific validation requirements for particular items. There was nothing smart, just a toil. The most useful thing was pattern matching on function parameters:
```haskell
isValidEntries::(String, String) -> Bool
isValidEntries ("ecl", s) = isValidEcl s
isValidEntries ("byr", s) = isValidByr s
isValidEntries ("iyr", s) = isValidIyr s
isValidEntries ("hcl", s) = isValidHcl s
isValidEntries ("eyr", s) = isValidEyr s
isValidEntries ("hgt", s) = isValidHgt s
isValidEntries ("pid", s) = isValidPid s
isValidEntries ("cid", _) = True
```
In "normal languages", you would need to write something like
```typescript
function isValidEntries(param1: String, param2: String): Bool {
  if (param1 == "ecl") {
    return isValidEcl(param2);
  }
}
```
or use a `switch` statement. In Haskell, you just repeat the function with a parameter value and define behaviour for that particular case.

_[Solution](https://github.com/miiila/aoc2020/blob/main/day4.hs)_


# Calling it a Days
I hope you have got some idea how I proceed when learning new things and how do I solve puzzles like these. This is just a preparation for more interesting problems and battle scars. I will not describe every single day - I will pick the most interesting (or painful :-)) ones to show you useful techniques, how functional programming can help to solve classical algorithmic problems. And also, how cumbersome it can be at the same time. Especially when you are learning it.



_Thanks for reading. Do you have any questions or errors to point out? Or just wanna chat? Let me know on [Twitter](https://twitter.com/MilaVot/status/1355568709532868614)._
