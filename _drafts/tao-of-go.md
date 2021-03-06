---
layout: 'post'
pusblished: false
title: "The Tao of Go"
---

Lots of people seem to like to critique [Go](http://golang.org) without fully understanding its origins or motivations. Most languages are designed with the aim of providing features that are useful to programmers. This sounds like a no-brainer, but Go was designed (it seems) with the aim of providing a simple langage that's easy to understand and reason about. Those two aims sometimes conflict.

> Why would you have a language that is not theoretically exciting? Because it’s very useful.

> -- Rob Pike

Lots of people complain about the seemingly "backwards" decisions Go made. "Why would you not implement paramatric polymorphism/generics?", "Why would you not allow custom iterables?", etc. etc. Go wasn't built to solve the kinds of problems that these things solve. Go was built to solve a different problem.

I think this is the most telling and informative account of Go's creation and why it is the way it is:

> In the span of an hour at that talk we heard about something like 35 new features that were being planned [for C++11]. In fact there were many more, but only 35 were described in the talk. Some of the features were minor, of course, but the ones in the talk were at least significant enough to call out. Some were very subtle and hard to understand, like rvalue references, while others are especially C++-like, such as variadic templates, and some others are just crazy, like user-defined literals.

> At this point I asked myself a question: Did the C++ committee really believe that was wrong with C++ was that it didn't have enough features? Surely [...] it would be a greater achievement to simplify the language rather than to add to it. 

> -- Rob Pike, [Less is Exponentially More](http://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html)

I think before you can truly understand Go you need to work with it a lot, where "working with it" entails not only writing Go but also _reading_ Go. It's easy to come up with clever abstractions that let you express complicated ideas in a small amount of code, but it's less clear how easily others will be able to understand what it is you're doing.

In Go, `range s` means exactly one of three things (namely, that `s` is a slice, map, or channel whose values we are iterating over). It's perhaps annoying that types like trees and linked lists can't leverage `range` to get nice syntax, but this helps simplyify the language, and makes it so that anyone who sees `range` knows immediately what's going on.

## Maps as sets

Go goes not have a built-in set type per-se, but a common idiom is to use a `map[T]bool` (or sometimes a `map[T]struct{}`) to represent a set of values.

    set := make(map[string]bool)
        
Inserting into a set looks like:

    set["foo"] = true
        
Checking for members looks like:

    if set["foo"] {
      fmt.Println("'foo' is in the set")
    }
        
Iterating looks like:

    for v := range set {
      fmt.Println(v)
    }

And deleting looks like:

    delete(set, "foo")
        
Some of this is a little clunky. `insert(set, "foo")` would be nice, and it might also be useful to have an explicit set type instead of having readers realize/know that a `map[string]bool` is a set.

But clunkiness asside, here we have a nice way of expressing the idea of set without having any additional language features or even libraries. All it takes to understand and reason about code involving "sets" is an understanding of maps.

## Broadcast events

Say you want to broadcast that an event has occurred to multiple interested parties. We can acheive this with a `chan struct{}`.

    event := make(chan struct{})
    
    for i := 0; i < 10; i++ {
    	go func() {
        	<-event
            fmt.Println("Got it")
        }
    }
    
    close(event)
    
By `close`ing the channel, we cause all readers to immediately read a zero value. They discard this value and continue on, knowing that the event has occurred.

`close` is perhaps an odd way of spelling "notify" and `<-event` an odd way of spelling "wait", but if we're willing to put up with that, we have a nice way of expressing level-triggered events using only channels.
