---
layout: single
title:  "Tabs vs Spaces"
date:   2013-11-24 22:05 +0000
categories: programming fsharp holy wars
author_profile: true
---
### Spaces, the final frontier
Tabs vs Spaces: the greatest of all the holy wars. Well it turns out that in F# there is no need for that debate: because in F# tabs are treated as an error by the compiler.

	let spaceFun x =
		x // ok
	
	let tabFun x =
		x // error!

Now if only there was an option for the F# compiler to treat nulls as errors too...