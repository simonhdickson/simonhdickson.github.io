---
layout: single
title:  "Ah fuget it"
date:   2014-08-16 20:16 +0000
categories: nuget fuget
author_profile: true
---

If you use the F# interactive alot like me, you've probably often come undone because there is no easy way to just pull down a nuget package and work with it. Often I end up creating a dummy project just to reference something via nuget and having to script there.

### fuget

Even though it sounds like "nuget" with an f instead of an n, it is actually named after a word I often say when using nuget (or atleast sounds like ;)).

It has no external dependencies, it is just a single file that you need to copy into your directory and then you're away.

Simple example of using it:

    #load "fuget.fsx"
    open Fuget
    fuget "FSharp.Data" Latest
    #r "fuget/FSharp.Data/lib/net40/FSharp.Data.dll"

    open FSharp.Data

    type fuget = JsonProvider<"""{"fuget":"hello world"}""">
    
You can also request an exact version using:

    fuget "FSharp.Data" (Version "2.0.9")

It works both from the interactive and from fsx files (even from inside FAKE scripts), and it available [here](https://github.com/simonhdickson/fuget).

However I only wrote it yesterday afternoon while drinking beer so I in no way take responsibility for any damage it may cause. It also has some issues currently (it doesn't handle dependencies, there is no way to query for packages so you have to just know which package you want).

Btw, it does run on Mac (and probably Linux too but I don't have a Linux VM setup on this machine):

![Fuget terminal](/assets/media/fuget/mac-terminal-screenshot.png)
