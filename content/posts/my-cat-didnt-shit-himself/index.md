---
title: "My cat didn't shit himself"
description: "And why it's relevant to your problem solving methodology"
date: 2024-07-08T17:09:12+10:00
---

I have two cats. One travels in the car reasonably comfortably, and the other... doesn't. With poor Woz I always say that only two out of three is a good trip. I'll let you work out what the three are.

<!--more-->

Today we got _zero out of three_. It's unprecedented. Apart from being immensely proud of the little trooper, the engineer in me was immediately curious: _Why this time? What changed?_

### Cause and effect

If Woz and I believe in a sane universe, then we must believe there is **some rational set of variables** that determine whether he does his daily deposit at 100km/hr on the Eastern Freeway or not.

Similarly, in software, it should generally hold that the software that we work with is operating deterministically. It is this premise that gives rise to two common head-scratchers that we face in the course of building software:
- it was working fine and now, unexpectedly, it isn't
- it wasn't working and now, unexpectedly, it is

The irony in these scenarios is that the challenge is in the _knowing_. If I knew _which variables have changed_ then it is often trivial to control them going forward to reliably produce the outcome I (and Woz, and the car) want.

### Woz as a function

Imagine we represent Woz as a function.

```golang
GonnaShit(bool hadBrekky, bool isMorning, bool hadMeds, bool hadPepTalk) bool {
    /* the lord only knows */
}
```

If I don't know the internal behaviours of my ~~cat~~ function then I need to intelligently search the input space to determine which inputs give me the desired outcome. _(It's `false`, team, we want it to be `false`)_

Going back to a software problem, the principle is the same. If you don't know the cause then you need to tweak inputs until you observe the desired change in behaviour. 

### Be methodical

You have to **introduce methodical change that allows you to disambiguate** the cause of the behaviour. Limit the degrees of freedom that you introduce together -- if you change two variables at once, even if you get the result you want, you still don't know which variable did it or if it was the combination of the two.

> Working in reverse, this is the logic that we apply to **good integration and deployment**: one PR, one change. Shipping isolated, atomic changes means that if something goes wrong, we can **disambiguate based on time** because we can pinpoint the change that caused a change in behaviour.
{ .note }

### Be organised

> "What about your version of `big-cheese-supreme`? Could it be that?"
> 
> "No, I already tried it with a `v2.x` of `big-cheese-supreme`. Oh, wait. Was that before or after I disabled the `thickCrusts` flag? Hmmm.
>
> -- every debugging session, ever

As you explore the input space, you will need to remain clear on the combinations that have and have not been tested. _Boolean logic_ offers the concept of a "**truth table**" for enumerating the logic of a given problem. A truth table for Woz's `GonnaShit` function might look like the following:

| `hadBrekky` | `isMorning` | `hadMeds` | `hadPepTalk` | **`didShit`** |
| ----------: | ----------: | --------: | -----------: | ------------: |
|         Yes |         Yes |       Yes |          Yes |       **Yes** |
|         Yes |         No  |       Yes |          No  |       **Yes** |
|         Yes |         No  |       No  |          Yes |       **Yes** |
|         ... |         ... |       ... |          ... |           ... |
|         ?   |         ?   |       ?   |          ?   |       **No** |

### Be patient

Woz and I don't yet know why he didn't shit himself this morning. I didn't think I changed anything: so maybe I messed up my truth table or maybe there is an input I haven't accounted for yet.

You might also get to the end of your truth table without a clear answer. It happens.

But one day, just like Woz, you might stop shitting yourself too. _(or solve that bug, whatever, I don't know your life)_

{{<figure src="./woz.jpg" title="What a bloody hero" >}}