---
layout: post
category: til
slug: go date format
title: Go does date formatting funny
---

I was recently trying to format the current date in go, and it does that in a way that felt really unintuitive to me at first but is actually
quite interesting.

## strftime-style formatting

If you're like me, you're used to the strftime-style string formatting where your format strings look like `"%Y %m %e"`. The thing is,
unless you have those codes totally ingrained in your head they're a bit weird to remember. `"%Y"` is the current year as a 4-digit number which is usually
what I want, but `"%y"` is just the last two digits. Whenever I have to work with that, I'm always going to websites like [For a Good Strftime](https://foragoodstrftime.com/#){:target="_blank"}.

## go's example-based formatting

Go does things totally differently--it wants you to give an example of the way you want the date formatted, and then it will figure it out from there.
The issue is, how do we fix ambiguity if we only have one example? `"12/10"` might be December 10 or October 12, and we don't inherently know.

So, here's go's solution to this problem. Just have the users give an example of a *known* date, which gets rid of ambiguity! If we know the user
is expressing the date December 10, `"12/10"` is always month/day formatting. We just have to choose what unambiguous date that will be.

## the example date in go

The date they chose is Monday Jan 2, 2006, at 3:04:05PM Mountain Standard Time. That seems totally out of left field, but there's actually a reason.
You can also read about this at the actual [time library page](https://pkg.go.dev/time#Layout){:target="_blank"}, but here's my best explanation.

The way this date is set up makes it so each field has a unique number. It's like this:

```
01  02  03  04   05   06   07

Jan 2nd 3pm 4min 5sec 2006 UTC-7
```

Tadaaaa! This is interesting because in order to work with the go date formatting system you have to think about the format
you want into an American-style format with month before day and a 12-hour format. This is a sort of weird choice in my eyes,
but it's really interesting to see things working in a way that I haven't seen before!
