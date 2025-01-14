Reflections
===========

<!--
This file generated by the build script at ./Build.hs from the files in
./reflections.  If you want to edit this, edit those instead!
-->

*[2016][]* / *[2017][]* / *[2018][]* / *[2019][]* / *2020*

[2016]: https://github.com/mstksg/advent-of-code-2016/blob/master/reflections.md
[2017]: https://github.com/mstksg/advent-of-code-2017/blob/master/reflections.md
[2018]: https://github.com/mstksg/advent-of-code-2018/blob/master/reflections.md
[2019]: https://github.com/mstksg/advent-of-code-2019/blob/master/reflections.md

[Available as an RSS Feed][rss]

[rss]: http://feeds.feedburner.com/jle-advent-of-code-2020

Table of Contents
-----------------

* [Day 1](#day-1)
* [Day 2](#day-2)
* [Day 3](#day-3)
* [Day 4](#day-4) *(no reflection yet)*

Day 1
------

<!--
This section is generated and compiled by the build script at ./Build.hs from
the file `./reflections/day01.md`.  If you want to edit this, edit
that file instead!
-->

*[Prompt][d01p]* / *[Code][d01g]* / *[Rendered][d01h]*

[d01p]: https://adventofcode.com/2020/day/1
[d01g]: https://github.com/mstksg/advent-of-code-2020/blob/master/src/AOC/Challenge/Day01.hs
[d01h]: https://mstksg.github.io/advent-of-code-2020/src/AOC.Challenge.Day01.html

So there's a simple-ish Haskell solution for these problems,

`tails` lets you separate out each item in a list with the list of items after
it:

```haskell
ghci> tails [1,2,3,4]
[1:[2,3,4], 2:[3,4], 3:[4], 4:[]]
```

```haskell
findPair :: [Int] -> Maybe Int
findPair xs = listToMaybe $ do
    x:ys <- tails xs
    y    <- ys
    guard (x + y == 2020)
    pure (x*y)

findTriple :: [Int] -> Maybe Int
findTriple xs = listToMaybe $ do
    x:ys <- tails xs
    y:zs <- tails ys
    z    <- zs
    guard (x + y + z == 2020)
    pure (x*y*z)
```

But this method is a little bit "extra", since we actually don't need to search
all of `ys` for the proper sum...if we pick `x` as `500`, then we really only
need to check if `1520` is a part of `ys`.

So we really only need to check for set inclusion:

```haskell
import qualified Data.IntSet as IS

findPair :: Int -> IS.IntSet -> Maybe Int
findPair goal xs = listToMaybe $ do
    x <- IS.toList xs
    let y = goal - x
    guard (y `IS.member` xs)
    pure (x * y)
```

And our first part will be `findPair 2020`!

You could even implement `findTriple` in terms of `findPair`, using `IS.split`
to partition a set into all items smaller than and larger than a number.
Splitting is a very efficient operation on a binary search tree like `IntSet`:

```haskell
findTriple :: Int -> IS.IntSet -> Maybe Int
findTriple goal xs = listToMaybe $ do
    x <- IS.toList xs
    let (_, ys) = IS.split x xs
        goal'   = goal - x
    case findPair goal' ys of
      Nothing -> empty
      Just yz -> pure (x*yz)
```

But hey...this recursive descent is kind of neat.  We could write a general
function to find any goal in any number of items!

```haskell
-- | Given a number n of items and a goal sum and a set of numbers to
-- pick from, finds the n numbers in the set that add to the goal sum.
knapsack
    :: Int              -- ^ number of items n to pick
    -> Int              -- ^ goal sum
    -> IS.IntSet        -- ^ set of options
    -> Maybe [Int]      -- ^ resulting n items that sum to the goal
knapsack 0 _    _  = Nothing
knapsack 1 goal xs
    | goal `IS.member` xs = Just [goal]
    | otherwise           = Nothing
knapsack n goal xs = listToMaybe $ do
    x <- IS.toList xs
    let goal'   = goal - x
        (_, ys) = IS.split x xs
    case knapsack (n - 1) goal' ys of
      Nothing -> empty
      Just rs -> pure (x:rs)
```

And so we have:

```haskell
part1 :: [Int] -> Maybe Int
part1 = knapsack 2 2020 . IS.fromList

part2 :: [Int] -> Maybe Int
part2 = knapsack 3 2020 . IS.fromList
```

And we could go on, and on, and on!

Definitely very unnecessary, but it does shave my time on Part 2 down from
around 2ms to around 20μs :)


### Day 1 Benchmarks

```
>> Day 01a
benchmarking...
time                 6.513 μs   (6.106 μs .. 6.965 μs)
                     0.971 R²   (0.965 R² .. 0.981 R²)
mean                 6.893 μs   (6.539 μs .. 7.138 μs)
std dev              1.093 μs   (965.5 ns .. 1.197 μs)
variance introduced by outliers: 94% (severely inflated)

* parsing and formatting times excluded

>> Day 01b
benchmarking...
time                 57.16 μs   (54.29 μs .. 59.82 μs)
                     0.982 R²   (0.975 R² .. 0.989 R²)
mean                 62.17 μs   (60.35 μs .. 64.08 μs)
std dev              6.463 μs   (5.113 μs .. 9.358 μs)
variance introduced by outliers: 84% (severely inflated)

* parsing and formatting times excluded
```



Day 2
------

<!--
This section is generated and compiled by the build script at ./Build.hs from
the file `./reflections/day02.md`.  If you want to edit this, edit
that file instead!
-->

*[Prompt][d02p]* / *[Code][d02g]* / *[Rendered][d02h]*

[d02p]: https://adventofcode.com/2020/day/2
[d02g]: https://github.com/mstksg/advent-of-code-2020/blob/master/src/AOC/Challenge/Day02.hs
[d02h]: https://mstksg.github.io/advent-of-code-2020/src/AOC.Challenge.Day02.html

Day 2, not too bad for Haskell either :)

There is some fun in parsing here:

```haskell
data Policy = P
    { pIx1  :: Int
    , pIx2  :: Int
    , pChar :: Char
    , pPass :: String
    }

parsePolicy :: String -> Maybe Policy
parsePolicy str = do
    [ixes,c:_,pwd] <- pure $ words str
    [ix1,ix2]      <- pure $ splitOn "-" ixes
    P <$> readMaybe ix1
      <*> readMaybe ix2
      <*> pure c
      <*> pure pwd
```

I used one of my more regular do-block tricks: if you pattern match in a
`Maybe` do-block, then failed pattern matches will turn the whole thing into a
`Nothing`.  So if any of those list literal pattern matches failed, the whole
block will return `Nothing`.

In any case, we just need to write a function to check if a given policy is
valid for either criteria:

```haskell
countTrue :: (a -> Bool) -> [a] -> Int
countTrue p = length . filter p

validate1 :: Policy -> Bool
validate1 P{..} = n >= pIx1 && n <= pIx2
  where
    n = countTrue (== pChar) pPass

validate2 :: Policy -> Bool
validate2 P{..} = n == 1
  where
    n = countTrue (== pChar) [pPass !! (pIx1 - 1), pPass !! (pIx2 - 1)]
```

And so parts 1 and 2 are just a count of how many policies are true :)

```haskell
part1 :: [Policy] -> Int
part1 = countTrue validate1

part2 :: [Policy] -> Int
part2 = countTrue validate2
```


### Day 2 Benchmarks

```
>> Day 02a
benchmarking...
time                 64.39 μs   (64.30 μs .. 64.56 μs)
                     1.000 R²   (1.000 R² .. 1.000 R²)
mean                 64.49 μs   (64.39 μs .. 64.65 μs)
std dev              419.1 ns   (229.4 ns .. 624.1 ns)

* parsing and formatting times excluded

>> Day 02b
benchmarking...
time                 78.82 μs   (77.60 μs .. 79.77 μs)
                     0.999 R²   (0.999 R² .. 1.000 R²)
mean                 79.02 μs   (78.43 μs .. 79.62 μs)
std dev              2.188 μs   (1.643 μs .. 2.975 μs)
variance introduced by outliers: 26% (moderately inflated)

* parsing and formatting times excluded
```



Day 3
------

<!--
This section is generated and compiled by the build script at ./Build.hs from
the file `./reflections/day03.md`.  If you want to edit this, edit
that file instead!
-->

*[Prompt][d03p]* / *[Code][d03g]* / *[Rendered][d03h]*

[d03p]: https://adventofcode.com/2020/day/3
[d03g]: https://github.com/mstksg/advent-of-code-2020/blob/master/src/AOC/Challenge/Day03.hs
[d03h]: https://mstksg.github.io/advent-of-code-2020/src/AOC.Challenge.Day03.html

Here I'm going to list two methods --- one that involves pre-building a set to
check if a tree is at a given point, and the other involves just a single
direct traversal checking all valid points for trees!

First of all, I'm going to reveal one of my favorite secrets for parsing 2D
ASCII maps!

```haskell
asciiGrid :: IndexedFold (Int, Int) String Char
asciiGrid = reindexed swap (lined <.> folded)
```

This gives you an indexed fold (from the *[lens][]* package) iterating over
each character in a string, indexed by `(x,y)`!

[lens]: https://hackage.haskell.org/package/lens

This lets us parse today's ASCII forest pretty easily into a `Set (Int, Int)`:

```haskell
parseForest :: String -> Set (Int, Int)
parseForest = ifoldMapOf asciiGrid $ \xy c -> case c of
    '#' -> S.singleton xy
    _   -> S.empty
```

This folds over the input string, giving us the `(x,y)` index and the character
at that index.  We accumulate with a monoid, so we can use a `Set (Int, Int)`
to collect the coordinates where the character is `'#'` and ignore all other
coordinates.

Admittedly, `Set (Int, Int)` is sliiiightly overkill, since you could probably
use `Vector (Vector Bool)` or something with `V.fromList . map (V.fromList .
(== '#')) . lines`, and check for membership with double-indexing.  But I was
bracing for something a little more demanding, like having to iterate over all
the trees or something.  Still, sparse grids are usually my go-to data
structure for Advent of Code ASCII maps.

Anyway, now we need to be able to traverse the ray.  We can write a function to
check all points in our line, given the slope (delta x and delta y):

```haskell
countTrue :: (a -> Bool) -> [a] -> Int
countTrue p = length . filter p

countLine :: Int -> Int -> Set (Int, Int) -> Int
countLine dx dy pts = countTrue valid [0..322]
  where
    valid i = (x, y) `S.member` pts
      where
        x = (i * dx) `mod` 31
        y = i * dy
```

And there we go :)

```haskell
part1 :: Set (Int, Int) -> Int
part1 = countLine 1 3

part2 :: Set (Int, Int) -> Int
part2 pts = product $
    [ countLine 1 1
    , countLine 3 1
    , countLine 5 1
    , countLine 7 1
    , countLine 1 2
    ] <*> [pts]
```

Note that this checks a lot of points we wouldn't normally need to check: any y
points out of range (322) for `dy > 1`.  We could add a minor optimization to
only check for membership if `y` is in range, but because our check is a set
lookup, it isn't too inefficient and it always returns `False` anyway.  So a
small price to pay for slightly more clean code :)

So this was the solution I used to submit my original answers, but I started
thinking the possible optimizations.  I realized that we could actually do the
whole thing in a single traversal...since we could associate each of the points
with coordinates as we go along, and reject any coordinates that would not be
on the line!

We can write a function to check if a coordinate is on a line:

```haskell
validCoord
    :: Int      -- ^ dx
    -> Int      -- ^ dy
    -> (Int, Int)
    -> Bool
validCoord dx dy = \(x,y) ->
    let (i,r) = y `divMod` dy
    in  r == 0 && (dx * i) `mod` 31 == x
```

And now we can use `lengthOf` with the coordinate fold up there, which counts
how many traversed items match our fold:

```haskell
countLineDirect :: Int -> Int -> String -> Int
countLineDirect dx dy = lengthOf (asciiGrid . ifiltered tree)
  where
    checkCoord = validCoord dx dy
    tree pt c = c == '#' && checkCoord pt
```

And this gives the same answer, with the same interface!

```haskell
part1 :: String -> Int
part1 = countLineDirect 1 3

part2 :: String -> Int
part2 pts = product $
    [ countLineDirect 1 1
    , countLineDirect 3 1
    , countLineDirect 5 1
    , countLineDirect 7 1
    , countLineDirect 1 2
    ] <*> [pts]
```

Is the direct single-traversal method any faster?

Well, it's complicated, slightly.  There's a clear benefit in the pre-built set
method for part 2, since we essentially build up an efficient structure (`Set`)
that we re-use for all five lines.  We get the most benefit if we build the set
once and re-use it many times, since we only have to do the actual coordinate
folding once.

So, directly comparing the two methods, we see the single-traversal as
faster for part 1 and slower for part 2.

However, we can do a little better for the single-traversal method.  As it
turns out, the lens indexed fold is kind of slow.  I was able to write the
single-traversal one a much faster way by directly just using `zip [0..]`,
without losing too much readability.  And with this *direct* single traversal
and computing the indices manually, we get a much faster time for part 1 (about
ten times faster!) and a slightly faster time for part 2 (about 5 times
faster).  The benchmarks for this optimized version are what is presented
below.


### Day 3 Benchmarks

```
>> Day 03a
benchmarking...
time                 334.7 μs   (320.0 μs .. 349.5 μs)
                     0.981 R²   (0.963 R² .. 0.991 R²)
mean                 335.4 μs   (321.5 μs .. 383.8 μs)
std dev              80.45 μs   (29.40 μs .. 163.8 μs)
variance introduced by outliers: 95% (severely inflated)

* parsing and formatting times excluded

>> Day 03b
benchmarking...
time                 1.611 ms   (1.590 ms .. 1.627 ms)
                     0.995 R²   (0.989 R² .. 0.998 R²)
mean                 1.567 ms   (1.529 ms .. 1.597 ms)
std dev              117.2 μs   (83.71 μs .. 167.7 μs)
variance introduced by outliers: 57% (severely inflated)

* parsing and formatting times excluded
```



Day 4
------

<!--
This section is generated and compiled by the build script at ./Build.hs from
the file `./reflections/day04.md`.  If you want to edit this, edit
that file instead!
-->

*[Prompt][d04p]* / *[Code][d04g]* / *[Rendered][d04h]*

[d04p]: https://adventofcode.com/2020/day/4
[d04g]: https://github.com/mstksg/advent-of-code-2020/blob/master/src/AOC/Challenge/Day04.hs
[d04h]: https://mstksg.github.io/advent-of-code-2020/src/AOC.Challenge.Day04.html

*Reflection not yet written -- please check back later!*

### Day 4 Benchmarks

```
>> Day 04a
benchmarking...
time                 62.34 μs   (56.91 μs .. 66.65 μs)
                     0.971 R²   (0.955 R² .. 0.991 R²)
mean                 62.00 μs   (60.08 μs .. 67.57 μs)
std dev              11.38 μs   (5.975 μs .. 19.65 μs)
variance introduced by outliers: 94% (severely inflated)

* parsing and formatting times excluded

>> Day 04b
benchmarking...
time                 1.353 ms   (1.268 ms .. 1.426 ms)
                     0.972 R²   (0.959 R² .. 0.984 R²)
mean                 1.293 ms   (1.247 ms .. 1.341 ms)
std dev              160.4 μs   (144.1 μs .. 182.0 μs)
variance introduced by outliers: 79% (severely inflated)

* parsing and formatting times excluded
```

