---
layout: post
title: "What Uncle Bob says about methods"
categories: code-style
---

# What Uncle Bob says about methods

In this article I will present a summary of one chapter of a book: ["Clean Code: A Handbook of Agile Software Craftsmanship"](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
written by Robert C. Martin also known as [Uncle Bob](https://en.wikipedia.org/wiki/Robert_C._Martin)
The chapter that I will summarize is: `Chapter 3: Functions`.

A summary can never be as good as the whole chapter or the book. If you still have not read this book, I highly recommend that you do it. If you did it in the mast, like me, it is good to refresh the knowledge and reflect on your practices.

## Keep it small
There is a discussion on how big a function should be. In the past, the rule was to fit into one screen. However, screens and resolutions have grown in recent years, so this should no longer be an indicator.
Uncle Bob says: `Functions should hardly ever be 20 lines long.`. Whereas in other videos he recommends even 4 lines per function. I find this criterion very difficult to fulfill. But the message is clear - `keep it small`.

## Do one thing
`Functions should do one thing. They should do it well. They should do it only.` This rule stipulates that functions should have only one responsibility. In many cases, it is difficult to define `one thing`. The answer is: `If a function does only those steps that are one level below the stated name of the function, then the function is doing one thing`.

## The Step-down rule.
This rule is about organizing the functions in the text file. The code should read like a top-down narrative. Start with high-level, abstract paragraphs, then dive into details.
First define a public method, below put private functions called from the public method. They should be put in an order of usage. This way it is easy to read.

## Switch Statements.
Avoid switch statements. It breaks the [Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle).
If you really, really cannot avoid `switch cases`, bury the switch case to a lower level class and never repeat it.

## Use descriptive names
Naming is extremely hard, but the time invested will be of benefit. Developers, or even future you, will read your code by names of variables, functions, classes, interfaces.
The names should read like a story, not something enigmatic. A long descriptive name is better than an enigmatic name.
By design, function names should be a verb (action).

# Function arguments.
The ideal number of arguments is `Zero`. If you need to use `one`, then `two`. Three arguments should be avoided. More is not acceptable. When a function seems to need more than two or three arguments, it is likely that some of those arguments ought to be wrapped into a class of their own.

## Output arguments
Avoid output arguments. Use the return value to replace the output arguments.

## Flag arguments
Avoid using flag arguments. By design, this means that the method does two things. Define two functions instead and call them properly.

## Side Effects
Your function should not have any side effects. It should have a single responsibility and the name should suggest what this responsibility is.

## Command Query Separation
`Functions should either do something or answer something, but not both`. You can read more about this in my [other article](https://marekhudyma.com/cqrs/2020/03/01/CQRS.html).

## Prefer Exceptions to Returning Error Codes
Returning error codes are a clear violation of `Command Query Separation`. When you have an unexpected situation (like no space on disk) throw an exception.

## Don't repeat yourself (DRY)
Never repeat the same code. If you use `ctrl-c`/`ctrl-v`, this is bad. If you need to change the logic in the future, good luck finding all the places with the code repeated.

# Conclusion
Uncle Bob provides a list of good advice on how to organize code in a way it is easy to read, understand, make changes to and maintain over a long term. Usually, it is the future of you or your colleagues. Be kind to these people!