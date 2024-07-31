---
layout: post
category: til
slug: bloom filter
title: How a Bloom Filter Works
---

While I was getting ready to come home from college this year, I was reminded
of a data structure I didn't know much about: **bloom filters**. (yes, it was
because of [the previous Friday's xkcd](https://xkcd.com/2934){:target="_blank"}.)

I previously had seen people use them in their own projects, but I'd never tried
one myself. So I decided to learn how they actually work, and what makes them
useful!

## Table of contents
- [Table of contents](#table-of-contents)
- [Surface-Level Usage](#surface-level-usage)
- [How to implement?](#how-to-implement)
- [Where do false positives come from?](#where-do-false-positives-come-from)

## Surface-Level Usage

A bloom filter is sort of like a hash set, where you can add items to it and check for containment. There's one big advantage they have, and a corresponding disadvantage.
- **small, fixed space** - A bloom filter never grows or shrinks the memory it uses,
   no matter how many items you add to it. Also, it doesn't store the items
   you add, but rather a bit array. This makes it pretty space-efficient.
- **false positives** - The more items you add, the higher the chance is that the bloom filter reports an item is contained, even if it's never actually been added.
   
## How to implement?

I read a lot of overviews like that one, but they didn't answer the question
I started off with: how do you implement something like this? I worked through [a bloom filter implementation on GeeksForGeeks](https://www.geeksforgeeks.org/bloom-filters-introduction-and-python-implementation/){:target="_blank"}, and I want to
describe what I found the most helpful.

The backing structure of a bloom filter looks something like this:
- A bit array of some fixed size
- Several hash functions
  - I'm being vague about the numbers here, because the optimal values are defined
    by formulas I still don't understand. See the GeeksForGeeks article for those.

Then, we can add to the filter like so:
1. Use the different hash functions to generate indices within the bit array. The more hash functions, the more indices.
2. For each index, set that position in the bit array to 1. We might have overlap with a previously added item, which is to be expected. (This is where false positives start to creep in.)

With that done, this is how we check if an item is contained in the filter:
1. Generate the same list of multiple indices as when we added, using the different hash functions.
2. Check the bit array at each index.
   1. If **any** of them are 0, we can be certain we never added this item.
   2. If **all** of them are 1, we probably have added this item before.

## Where do false positives come from?

False positives crop up when we are checking for containment. Consider a really simple toy example, where our bit array is of length 10 and we have 3 hash functions.

First, let's add some items to our array. Let's pretend that "hello" hashes to 
[0, 7, 3]. Then, when we add it our bit array will contain the values

```text
bits:  1 0 0 1 0 0 0 1 0 0
i:     0 1 2 3 4 5 6 7 8 9
```

Then, let's add "world" which hashes to [2, 5, 1]. Now, our bit array holds

```text
bits:  1 1 1 1 0 1 0 1 0 0
i:     0 1 2 3 4 5 6 7 8 9
```

At this point, our array is starting to get pretty saturated. This is a bad sign,
and it means we're going to start seeing more and more false positives. 

Imagine we want to check whether "word" is contained, and it hashes to [0, 1, 2]
If we look in the bit array at these indices, we see that they are all 1. That means
we return that "word" is already contained, even though it's not.

On the bright side, we never get a false negative (which would mean we add an item,
but the filter says it is not contained). This is because all the hashed indices must be
zero for our containment function to return false, and that will never happen since
an item will always hash the same way.
