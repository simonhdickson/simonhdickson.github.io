---
layout: single
title:  "The Simon Says Monad"
date:   2013-11-22 15:56 +0000
categories: functional fsharp monad
author_profile: true
---

### What is a Monad?
Everyone tries to explain monads in a simple way and makes it sound way more complicated than it really is. Something is monadic if there are rule or set of rules that can define how a set of functions is combined into another function. That not strictly mathmatically true, but anything that fits that description could at least be described as monadic.

You might be reading that and thinking "surely everything is monadic?", well yes, almost everything is kind of monadic (or can be described in that way). 

### The Simon Says Monad
May I present the single most useful monad in the world!

    let doIt this that =
        simonSays{
            let! i = this
            let! j = that
            printfn "%i" <| i + j
        }

So, the way I see it, when you declare simonSays { ... }; you're basically saying "we don't play by normal rules: now we play by simonSays rules." What this means is, if you don't say "Simon Says", then you don't do anything. So for example, if we invoke it like this:

    doIt (SimonSays 1) (SimonSays 1)

Then the result is SimonSays 2. However if we do:

    doIt (SimonSays 1) SimonDidntSay

The the result is SimonDidntSay. To go back to the definition of monads earlier, "Something is monadic if there are rule or set of rules that can define how a set of functions is combined into another function". In the Simon Says monad, there is a single very simple rule: you must say "Simon Says", otherwise we don't do anything. If we re-write this code slightly to remove all the F# sugar, it become far more obvious what is happening here.

    let doIt this that =
        match this with
        | SimonSays i ->
            match that with
            | SimonSays j -> printfn "%i" <| i + j
            | _ -> ()
        | _ -> ()

If you build up a large chain of these things, then writing all this would get cumbersome.

### Maybe monad
If you know your monads, you may have already noticed that the "Simon Says" monad bears a slightly resemblance to the "Maybe" monad. Ok, it IS actually maybe monad. Infact you can find more information about the maybe monad [here](http://en.wikibooks.org/wiki/F_Sharp_Programming/Computation_Expressions).