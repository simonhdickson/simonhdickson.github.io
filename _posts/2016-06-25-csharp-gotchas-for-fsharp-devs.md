---
layout: single
title:  "C# gotchas for F# devs"
date:   2016-06-25 11:41:08 +0000
categories: fsharp csharp
author_profile: true
---

Like many F# developers I have arrived in F# via C#. Sometimes
when you go back to C# though there are traps that you can fall into, 
here a few that I find myself falling into.

### Implicit type conversion

In F# the following would error, since strings and ints are different types.

    // The type 'string' does not match the type 'int'
    let result = 1 + "1"

But in C# there is no error, it just continues anyway.

    [lang=cs]
    var result = 1 + "1"
    // result = "11"

### Type declarations required

In F# most type declarations are optional, so long as the compiler 
can work it out.

    let dict = Dictionary()
    dict.Add("key", "value")

But if you try this in C#, you'll get a massive error. So you need to
provide the type declarations regardless, there are a few cases where 
you don't have to. But generally you'll have to get used to writing it
explictly.

    [lang=cs]
    var dict = new Dictionary<string, string>()
    dict.Add("key", "value")

### Lack of expressions

If we want to assign a variable as the result of a conditional 
it is easy.

    let port =
        if isProduction then
            80
        else if isStaging then
            81
        else
            5000

    // Same goes for pattern matching
    let port =
        match settings with
        | Production -> 80
        | Staging -> 81
        | _ -> 5000

But in C# this is not possible because conditions are statments
not expressions.

    [lang=cs]
    // I find myself trying to write code like quite a lot
    var port =
        if (isProduction)
            80;
        else if (isStaging)
            81;
        else
            5000;

Instead you have to write it using assignments.

    [lang=cs]
    int port;
    if (isProduction)
        port = 80;
    else if (isStaging)
        port = 81;
    else
        port = 5000;
        
This also has the disadvantage that if you don't have an else
statement you won't know (you do get a warning for reference types
but not for value types).

    let port =
        if isProduction then
            80
        else if isStaging then
            81
        // Error!

    // Same goes for pattern matching
    let port =
        match settings with
        | Production -> 80
        | Staging -> 81
        // Error!

But in C#...

    [lang=cs]
    int port;
    if (isProduction)
        port = 80;
    else if (isStaging)
        port = 81;
    // Did we mean to do this?

### Out parameters are ugly

F# handles out parameters in a suprizely slick way

    let result =
        match Int32.TryParse "1" with
        | true, i -> i
        | false, _ -> 2
    
I like this in particular because 'i' is only available if
it has a value.

    [lang=cs]
    int result;
    int i;
    if (int.TryParse("1", out i))
    {
        result = i;
    }
    else
    {
        result = 2;
    }

We're left with this floating 'i' here, that is really useless
unless we know the result of the TryParse method. This is slightly
more akward than above as because out parameters are always assigned
(even for reference types) so we won't even get an error if we try to use
it later by mistake.

### No errors on unused results

In F# and C# it isn't uncommon to return a status code, to state
what the result of a operation was. In F#, if we don't do anything
with the result, the compiler will let me know that we forgot
to handle it.

    type Result = Success | Failure

    let performOperation () =
        // Do something
        Failure
    
    let calculation () =
        // Error This expression should have type 'unit'
        performOperation ()
        Success

    let calculation () =
        // We can still choose to live dangerously though
        performOperation () |> ignore
        Success

So in F# by default it'll give us a hint that we might have 
introduced a bug there, but we can still ignore it and forget
about it if we wish.

In C# though, you get no warnings or errors.

    [lang=cs]
    enum Result {
        Success,
        Failure
    }

    Result PerformOperation()
    {
        // Do something
        return Result.Failure;
    }

    Result Calculation()
    {
        // Unless we remember to check here, the compiler
        // will never give us a hint
        PerformOperation();
        return Result.Success;
    }

These are just some of the things to be warying of when going
from F# to C#. There are some others gotchas related to async
but [that is a blog post in it's own right](http://tomasp.net/blog/csharp-async-gotchas.aspx).