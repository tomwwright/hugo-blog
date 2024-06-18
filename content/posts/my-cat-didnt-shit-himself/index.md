---
title: "My cat didn't shit himself"
description: "And why it's relevant to your software methodology"
date: 2024-06-17T17:09:12+10:00
draft: true
---

This morning, Woz didn't shit himself in the car. I had no idea why. The first thing I thought was: "This is just like software."

I have two cats. One travels in the car reasonably comfortably, and the other... doesn't. With poor Woz I always say that only two out of three is a good day. I'll let you work out what the three are.

Today we got _zero out of three_. It's unprecedented. Apart from being proud of the little trooper, the engineer in me was immediately curious: _Why this time? What changed?_

<!--more-->

### Cause and effect

If Woz and I believe in a sane universe, then we must believe there is **some rational set of variables** that determine whether he does his daily deposit at 100km/hr on the Eastern Freeway or not.

The same is true of our software woes. Two common head-scratchers in software:
- it was working fine, and now it isn't
- it wasn't working, and now it is

The irony in these scenarios is that often the variables are easy to control. If I knew _which ones have changed_ then generally I can control them to reliably produce the outcome I (and Woz, and the car) want.

### Woz as a function

Imagine we represent Woz as a function.

```golang
GonnaShit(bool hadBrekky, bool isMorning, bool hadMeds, bool hadPepTalk) bool {
    /* the lord only knows */
}
```

If I don't know the behaviours of my function then I need to intelligently search the input space to determine which ones give me the right outcome. _(It's `false`, team, we want it to be `false`)_

### Be methodical

You have to **introduce intentional change that allows you to disambiguate** the cause of the behaviour you are affecting. Limit the degrees of freedom that you introduce -- if you change two variables at once, even if you get the result you want, you still don't know which variable did it or if it was the combination of the two.

> This is the logic that we apply to **good deployment practices**: _one PR, one change_. Shipping isolated changes means that if something goes wrong, we can **disambiguate by time** alone without expensive investigation.
{ .note }

### Be organised

> "What about your version of `big-cheese-supreme`? Could it be that?"
> 
> "No, I already tried it with a `2.x` of `big-cheese-supreme`. Oh, wait. Was that before or after I disabled the `thickCrusts` flag? Hmmm.
>
> -- every debugging session, ever

As you explore the input space, you will need to remain clear on the combinations that have and have not been tested. _Boolean logic_ offers the somewhat stuffy concept of a "**truth table**" for enumerating the logic of a given problem -- a truth table for Woz's `GonnaShit` function might look like the following.

| `hadBrekky` | `isMorning` | `hadMeds` | `hadPepTalk` | **`didShit`** |
| ----------: | ----------: | --------: | -----------: | ------------: |
|         Yes |         Yes |       Yes |          Yes |       **Yes** |
|         Yes |         No  |       Yes |          No  |       **Yes** |
|         Yes |         No  |       No  |          Yes |       **Yes** |
|         ... |         ... |       ... |          ... |           ... |
|         ?   |         ?   |       ?   |          ?   |       **No** |

### Be patient

Woz and I don't yet know why he didn't shit himself this morning. Maybe I messed up my truth table or maybe there is a factor I haven't accounted for yet.

You might also get to the end of your truth table without a clear answer. It happens. Check your work, then have a cup of tea and a think on the missing variable. You'll find it.

One day, just like Woz, you might stop shitting yourself too. _(or solve that bug, whatever, I don't know your life)_

{{<figure src="./woz.jpg" title="What a bloody hero" >}}