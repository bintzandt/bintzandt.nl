---
title: "A closer look at Javascript: bizarre examples"
date: "2019-10-23T20:25:00.169Z"
template: "post"
draft: false
slug: "/posts/writing-banana-in-js/"
category: "Development"
tags:
  - "JavaScript"
  - "Fun"
description: "
JavaScript is a very flexible language.
Some people love that while others really hate it.
This post shows how you can abuse JavaScript's flexibility for some weird stuff.
"
---
- [Banana power](#banana-power)
- [From int to string](#from-int-to-string)

A lot can be said about JavaScript.
No matter what, it is clear that JavaScript is taking over the internet.
Some years ago, JS was only used in the front-end for making websites dynamic.
Today, most websites won't even render without it.

Not to mention all the back-end JavaScript frameworks that seem to pop-up like mushrooms.
Most of them are based on [ExpressJS]([https://expressjs.com/]) and [NodeJS](https://nodejs.org/en/).

Lucky for us, JavaScript has changed a lot as well.
Modern JavaScript has a lot of features that we expect from a high-level programming language.
Think of the `map`, `reduce` and `filter` functions that make it possible to easily loop over arrays.
And don't forget that Functional Programming is possible with JS as well ([1]).

But although JS has changed a lot for the better, there is still enough functionality that makes you wonder what is happening under the hood.

This post guides you through what I think are some of the most bizarre examples of the "power" of JavaScript.

## Banana power
Try to guess what the following code will output:
```javascript
( "b" + "a" + + "a" + "a" ).toLowerCase()
```
Okay, that was not that hard. The title spoils everything. But now try to explain why this happens.

That's a bit harder. 
But not too hard, right?
It has to do with the `+ + "a"`.

If we write that a little different we get `+ +"a"`.
`+"a"` is JavaScript syntax for converting the string `a` to a number.
For example, `+"1"` will output the *integer*&nbsp;`1`.

However, JS does not know how to convert `a` to a number.
This results in a special JavaScript result `NaN`, which has type `number`.

For most numbers, it makes sense that we can add them to a string without having to explicitly convert it to a string first.
This is not the case for `NaN`. But for some reason the developers of JavaScript have kept this behaviour the same.
Thus allowing us to concat `NaN` to string by implicitly converting it to the string `"NaN"`.
The complete result of all the additions is therefore `baNaNa`.
When we convert that to lowercase using a standard JavaScript function, we get our result: `banana`.

## From int to string
Imagine that you have this array of strings `[ "0", "1", "111" ]`.
You want to convert all the items in this array to an integer.
Seems reasonable easy right?

Well, as it turns out: this can go wrong in JavaScript.

It is easy to do this using the `Array.map()` syntax (look [here][2] for more information).
This results in the following code:
```javascript
[ "0", "1", "111" ].map( parseInt ) 
```
However, the code does not do what you expect it to do. This code will output `[ 0, NaN, 7 ]`. So why is that?

It turns out that `parseInt` wants to receive two parameters([3]): a string to parse and a radix to parse the number in.
When we look at the documentation of `Array.map()` we can find out that it actually provides three parameters to the function.
The third one is simply ignored.
The parameters are: the element of the array, it's index and the complete array.
Below is a table trace of the execution of this program.

| element | index | function call | result |
|:-------:|:-----:|:-------------:|:------:|
|  `"0"`  | `0` | `parseInt( "0", 0 )` | `0` |
|  `"1"`  | `1` | `parseInt( "1", 1 )` | `NaN` |
|  `"111"`  | `2` | `parseInt( "111", 2 )` | `7` |

In `parseInt` function, a radix of `0` is the same as radix `10` so that is simply converting the string to our number system.
A radix of `1` makes no sense, so that is why the output of the function is `NaN` in that case.
And as most of you know, `111` is the binary representation of `7`. So `parseInt` of `"111"` with radix `2` is `7`.

This explains the "weird" output that the program gives and if you think about it: it is not weird at all.
It is just not something that you would expect.

A possible solution for this "problem" would be to make explicit how many parameters you want to use.
This is demonstrated in the code below.

```javascript
[ "0", "1", "111" ].map( numberString => parseInt( numberString, 10 ) )
```

JavaScript is a fun language.
You can do things with it that are not even possible in other languages.
This also has a flipside: bugs are often hard to track and some seemingly normal code can behave completely unexpected.

That is why I would recommend using [TypeScript](https://www.typescriptlang.org/).
It's a superset of JavaScript developed by MicroSoft.
This is a best of both worlds: it allows you to add types and type checking to JavaScript while still allowing the flexibility if you really need it. 

[1]: https://opensource.com/article/17/6/functional-javascript
[2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt
