---
layout: single
title:  "Nancy + F# = Fancy"
date:   2013-12-14 10:00 +0000
categories: fsharp fancy nancy
author_profile: true
---
Last week I made the [owin monad](blog/2013/12/03/owin-the-myth), and when I showed it to [@@nbevans](https://twitter.com/nbevans) he suggested making it work with nancy.

So I did.

Let me introduce Fancy:

    let pipeline =
        fancy {
            get "/" (fun () -> sprintf "Hello World!")
            get "/%s" (fun name -> sprintf "Hello %s!" name) 
            get "/square/%i" (fun number -> sprintf "%i" <| number * number) 
        }

I won't dig too heavily into the code at this moment, it is quite ugly and hacky in places. It is also obviously not production ready. However, the nice thing about this is that it is fully typed, i.e:

    let pipeline =
        fancy {
            get "/%s" (fun name -> sprintf "%s" name)       // Works
            get "/%s" (fun (name:int) -> sprintf "%s" name) // Compiler error!
            get "/%i" (fun name -> sprintf "%s" name)       // Compiler error!
            get "/%s" (fun name -> sprintf "%i" name)       // Compiler error!
        }

Now if you know your nancy, you'll know that nancy doesn't expect the format "/%s" or "/%i"; it expects the format "/{name}" or "/{number:int}". And you probably also know that all normally StringFormat functions in F# take type parameters, rather than a function of those types.

Well what happens is we're only using StringFormat for static type checking, we're not actually using StringFormat here quite as it was intended.

    member this.Get(m, url:StringFormat<'a->'b,'c>, processor:'a->'b) =
        // ommitted

These type constraints do that. There is actually a slight error, with the above code, but I'm reasonably certain it can be converted into a feature. You see you are required to have the correct parameters in the correct order, but extra paramters are allowed.

    let pipeline =
        fancy {
            get "/%s" (fun name (other:int) -> sprintf "%s" name)       // Compiles
        }

Currently this will cause the code to fail, however it is quite possible we can use that to implement a convention based system for passing extra things like headers, parameters, etc. 

Anyway here is a [link to the repo](https://github.com/simonhdickson/Fancy), feel free to grab the code and have a play!