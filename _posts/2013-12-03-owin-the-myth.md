---
layout: single
title:  "Owin: The Myth, the Monad"
date:   2013-12-03 20:37 +0000
categories: owin fsharp monad
author_profile: true
---
I've played with owin in F# [before](https://github.com/simonhdickson/Bar) and recently have been playing with computational builders. So I thought the other day, how about combining them both so you can write something long the lines of this:

    let pipeline =
        owin {
            get "/" <@@ fun () -> "Hello World" @@>
        }

Gotta love that syntactic sugar, so how do we do this? Well really, the owin monad is actually really simple to implement. You have a state, in this case the state is the is the IAppBuilder. So for posterity let's just steal the important bits of the state monad from FSharpx and use that. So let's skip to the good bit:
	
    type OwinBuilder() =
        // omitted
        [<CustomOperation("get", MaintainsVariableSpaceUsingBind=true)>]
        member this.Get(m, url, processor) =
            this.Bind(m, fun a ->
                this.Bind(getState, fun (app:IAppBuilder) ->
                    putState <| app.UseRequestProcessor("GET", url, processor)))

In a computation expression there is a [standard workflow](http://msdn.microsoft.com/en-us/library/dd233182.aspx) for operations that you can use (this includes the Bind operation). In our case we don't actually want this, we want to define our own operators (for get, post, etc). The CustomOperation attribute allows you to define a these, and also describe how your custom operator works.

In this case we want to define our operator so that each line defines a new endpoint of our web api, and pass all the state around in the background. To do this the "MaintainsVariableSpaceUsingBind", works to bind all the statements together. In the above code any previous operators will be passed in the m paramter, and then the we're basically appending our new owin endpoint to end of that, which will then be passed into the next operator, etc.

Anyway, here is a [link to the repository](https://github.com/simonhdickson/OwinBuilder), if you're interested in looking into it further.