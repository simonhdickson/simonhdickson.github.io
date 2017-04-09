---
layout: single
title:  "Intro to Rust for .NET Developers"
date:   2017-04-09 08:18:39 +0000
categories: rust fsharp csharp dotnet
author_profile: true
---

Rust has been around for a few years now and it has started to become quite popular. If you're a C# or F# developer who hasn't really had time to read into it I just want to give you an idea what all the fuss it around it. I'm just going to show you what I consider to be the main three differences if you're coming from a .NET background.

This post isn't meant to convince you that you should use it. Rust is a primarily designed as a systems language, and while it has many features common in managed higher level programming languages in reality it sits a lot closer to C than C# or F#. If you're developing a video game or a database then it would be of interest to you, if you're implementing business domain logic you'll be unlikely to see the benefits.

I'm also going to mix and match C# and F# code in this blog post. Not for any particular reason, just because that's how I roll.

## Ownership

By far the biggest difference between Rust and almost every other language out there is Rust's ownership system. IF you don't understand how it works, you'll be hitting your head against your keyboard for hours wondering why code that looks perfectly reasonable doesn't compile. Let's start by looking at some F# code.

```ocaml
type MyType() = {
    int id;
}

let helloWorld = { id = 1 };

let helloWorld2 = helloWorld;

printfn "id is %i" helloWorld2.id;
```

In C# this is perfectly legal and if you run it, it will print `Hello World` as you would expect.

However if we now convert this code to Rust:

```rust
struct MyType {
    id:i32
}

fn main() {
    let hello_world = MyType { id: 1 };

    let hello_world2 = hello_world;

    println!("id is {0}", hello_world.id);
}
```

You'll get a rather strange looking compiler error that says something like `error: use of moved value: 'hello_world.id'`.

Why is this illegal? Well one of the Rust's rules is that there should only ever be one reference to a given resource. When we created the new reference to the instance of `MyType` we passed ownership of the underlying object to `hello_world2`. This means that when we tried to use it again later we can't, since we no longer own that resource.

There are several ways around this, one of the simplest is for the object to implement `Copy`:

```rust
#[derive(Copy, Clone)]
struct MyType {
    id:i32
}

fn main() {
    let hello_world = MyType { id: 1 };

    let hello_world2 = hello_world; // This line now creates a copy of MyType

    println!("id is {0}", hello_world.id);
}
```

In this case the `i32` type already implements `Copy` so we don't need a custom implementation. Now we get a similar experience to .NET's value types, there are other ways around this problem too like making the resource safe accessible across threads.

At this point you're probably wondering what the point is of all this, so I'll give a little example in C# of where this could come in handy:

```csharp
// Imagine this is in a library somewhere
public void DoStuff(IDbConnection connection)
{
    Task.Run(() => {
        while (true) {
            connection.Query("...");
        }
    });
}

var connection = ...;
var i = connection.Query("...");
DoStuff(connection);
var j = connection.Query("...");

```

In the .NET world writing a function something like this would be considered bad since the user of the library has no way of knowing we were going to keep using that resource (the IDbConnection) after the call returned. But you could say the problem is .NET has no way define who actually owns the object. What happens in Rust?

```rust
fn DoStuff(connection: IDbConnection) {
    thread::spawn(|| {
        while true {
            connection.Query("...");
        }
    });
}

fn main() {
    let connection = ...;

    let i = connection.Query(...);
    DoStuff(connection);
    let j = connection.Query(...);
}
``` 

If we compile this code, it'll fail and maybe not for the reason you would think at first. The compiler will error on the line where `j` is declared with an error like `error: use of moved value: 'connection'`. What is happening here is that when we called `DoStuff` we passed ownership of the `connection` to that function. That means that we're no longer allowed to use it anymore. Now that probably isn't what we wanted, what we probably wanted was to just lead the connection to the method:

```rust
fn DoStuff(connection: &Connection) {
    thread::spawn(|| {
        while true {
            connection.Query("...");
        }
    });
}

fn main() {isn't something that is
    let connection = ...;

    let i = connection.Query(...);
    DoStuff(connection);
    let j = connection.Query(...);
}
```

By adding the `&` to the definition we've told the compiler that this function only borrows this connection. So now when the function returns we're taking ownership of the connection back and the declaration of `j` is now valid. However now we'll get a different error, since the `DoStuff` function tries to keep the reference alive after the return of the function (which is when its borrow ends).

As you can see this allows functions to better describe how they operate on and can prevent errors which could otherwise be missed.

## Traits

Traits are are an important concept in Rust, they aren't unique to Rust but doesn't exist in any of the main .NET languages so is worth talking about still.

In Rust traits exist somewhere between attributes, extension methods, interfaces and in many ways even more than that. 

```ocaml

```

## 
