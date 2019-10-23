---
title: "Great Answers From StackOverflow #1"
date: 2019-10-23T07:12:52-03:00
draft: false
authors: ['mauricio']
tags: ['python', 'stackoverflow']
---
[This post](https://stackoverflow.com/questions/2489669/function-parameter-types-in-python) provided an excellent explanation about Python's strong but dynamic typing. Python typing classification confused me much more times than I'd like to admit, and I still see people having trouble with it nowadays. It's worth the 45 seconds reading and a couple minutes thinking.

---

## [Question: Function parameter types in Python](https://stackoverflow.com/questions/2489669/function-parameter-types-in-python)

Unless I'm mistaken, creating a function in Python works like this:

{{< highlight python>}}
def my_func(param1, param2):
    # stuff
{{</ highlight>}}

However, you don't actually give the types of those parameters. Also, if I remember, Python is a strongly typed language, as such, it seems like Python shouldn't let you pass in a parameter of a different type than the function creator expected. However, how does Python know that the user of the function is passing in the proper types? Will the program just die if it's the wrong type, assuming the function actually uses the parameter? Do you have to specify the type?

## Answer:

Python is strongly typed because every object _has_ a type, every object _knows_ its type, it's impossible to accidentally or deliberately use an object of a type "as if" it was an object of a _different_ type, and all elementary operations on the object are delegated to its type.

This has nothing to do with __names__. A _name_ in Python doesn't "have a type": if and when a name's defined, the name refers to an _object_, and the _object_ does have a type (but that doesn't in fact force a type on the _name_: a name is a name).

A name in Python can perfectly well refer to different objects at different times (as in most programming languages, though not all) -- and there is no constraint on the name such that, if it has once referred to an object of type X, it's then forevermore constrained to refer only to other objects of type X. Constraints on __names__ are not part of the concept of "strong typing", though some enthusiasts of __static__ typing (where names do get constrained, and in a static, AKA compile-time, fashion, too) do misuse the term this way.

---

I may be overreacting, but I think it is simple, clear and kinda brilliant.
