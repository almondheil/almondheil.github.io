---
layout: post
category: til
slug: average coin
title: The Average Value of a Coin
---

For reasons that are too stupid to write down, I decided to try and figure out
what *the average US coin* is. I love a good stupid statistical adventure, especially when I should be sleeping instead of working on a spreadsheet.

First, let me shoot down the easiest approach. We *could* just take the average of
1, 5, 10, and 25 cents. If we did this we'd get 10.25¢, but there's a factor I find more
interesting: the actual number of each coin that gets produced. In other words, I want
an average that's weighted by the number of coins produced.

To do this, I need some source on how many coins actually get minted each year.
Luckily, Wikipedia swoops in with a table showing coin production since the year 1887! (<https://en.wikipedia.org/wiki/United_States_Mint_coin_production>{:target="_blank"})

Copy+pasting this into Google Sheets, I was really surprised that it automatically
formatted everything correctly--I was worried the whole table would get pasted into a single cell,  which would be stupid. 
This made my job with data cleaning very easy--I just had to delete citations to make everything
recognized as a number.
absolute best they're gonna get is actual pants
At this point, I also decided that I wanted to find the average mass of a coin
so I quicky searched around for how much coins weighed.
I also decided to limit my date range to 1982 and later, 
because the mass of a penny varied way more before that time.

| Coin | Value ($) | Mass (g) |
------ | --------- | ---------- |
| Penny | 0.01 | 3.11[^1] |
| Nickel | 0.05 | 5 |
| Dime | 0.10 | ~2.2 |
| Quarter | 0.25 | ~5.7 |

> NOTE: The Wikipedia table also tracks half dollars and dollar coins, but I'm choosing
> to ignore these because I don't really care about them.

With this, I can sum up the number of coins produced since 1982, and use that to derive
the total value and total mass that have been minted.

| Coin | # Produced | Total Value ($) | Total Mass (g) |
------ | --------- | ---------- | -------------------- |
| Penny | 3.87e+11 | 3.87e+09	| 1.20e+12 |
| Nickel | 5.23e+10 | 2.62e+09 | 2.62e+11	|
| Dime | 8.98e+10 | 8.98e+09 | 1.97e+11 |
| Quarter | 8.45e+10 | 2.11e+10 | 4.82e+11 |

Two main things to note from this table:
- These are MASSIVE numbers! There have been 3.7 **billion** dollars of pennies produced alone.
- The most-produced coin is pennies, by a whole order of magnitude. I didn't expect this,
  I feel like I see higher denominations more often! Maybe I just don't notice pennies.

At this point, I can actually answer my initial question by taking these averages:

```text
                      sum(values)
average coin value = ------------- = $0.06
                      sum(number)
```

and 

```text
                     sum(masses)
average coin mass = ------------- = 3.5g
                     sum(number)
```

There we go, all done!

The average coin in the US since 1982 weighs 3.5 grams and is worth 6 cents--a slightly lighter and more valuable nickel.

This is interesting to me because it shows that the massive number of smaller denomination coins---particularly pennies---drags
the average way towards the lower denomination data, both in terms of value and mass.

---
{: data-content="footnotes"}

[^1]: Since 1982, the weight of a penny has either been 3.11 grams (for copper-plated zinc coins) or 2.5 grams (for brass ones) according to <https://cointrackers.com/blog/43/how-much-does-a-penny-weigh/>{:target="_blank"}. I'm sticking with the 3.11 gram measurement because I want to simplify some.
