---
layout: post
title: "Type checking in JS, an experiment"
photo: type-check.jpg
meta: post
---

![](/images/type-check.jpg)

There's a saying I've heard that involves developers and straight jackets. One might wear one and say "it's so constricting, I can't breathe or move!", while another will wear the same one and say "Ohh, feels so warm and cozy!". That's static typed languages for you.

Anyone who has ever worked with a statically typed language like Java must feel so out of sorts in Javascript. <!--more-->As such, there are all kinds of libraries out there to check types for you, even entire [new languages][dart] that compile to js have looked to solve this problem. Facebook [created an entire library][flow] to add their own annotations that do post processing for checking types. Obviously this is an issue people care about.

But javascript is a very expressive and dynamic language that should be able to handle something like this with no problem. So that's what I set out to do, because more or less, I'm a nerd that is interested in this sort of thing.

## The Basics

Usually, we care most about what goes in and what comes out of a function. Asserts can be used to help mediate this and aid in debugging, especially with live data. But this can be tedious, repetitive, and cumbersome. What I wanted was something simple, short and especially readable. A small library called [Typist][typist] is what I came up with and here is what it looks like:

{% highlight javascript %}
var makeArray = typist.takes(Array, String, Number)
                      .does(function(input, foo, bar) {
                        input.push(foo, bar);
                        return input;
                      })
                      .returns(Array)
                      .done();

makeArray([], "1", 2); // ["1", 2]
makeArray([], 1, 2); // Throws TypistError
{% endhighlight %}

I think just by reading it you can tell what is going on and that is the point, add type checking while maintaining a readable structure that is obvious what its purpose is.

Typist makes use of some other methods in its API for type checking: `typist.check()` which takes an array of types and values to check, and `typist[type]()` which takes a value that will be returned only if it is the right type.

If you were to write out the previous example literally and with those two methods, it would be the same as something like this:

{% highlight javascript %}
var makeArray = function(input, foo, bar) {
  typist.check([Array, input], [String, foo], [Number, bar]);

  input.push(foo, bar);
  return typist.array(input);
};
{% endhighlight %}

There's a bit more though, as another option is to use `typist(Type, function)` which will return a curried function, that when invoked, will have its return value type checked. If it is the right type, it gets returned and if not, a TypistError is thrown.

Here are some more examples that allow you to pick and choose the level of type checky-ness you want, the chainable all-in-one example from above to a more granular approach:

{% highlight javascript %}
var lowercase = function(str) {
  return typist.string(str).toLowerCase(); // returns the string if it is the right type
}
lowercase(1); // Throws TypistError

var lowercase = function(str) {
  if (typist.is.string(str)) {
    return str.toLowerCase();
  } else {
    // do something else
  }
}
lowercase(2); // will hit else branch

var lowercase = typist(String, function(str) {
  return null;
}
lowercase(3); // Throws TypistError
{% endhighlight %}

## Handling Errors

It would make the most sense to return a `TypeError` when a check fails, but this would be confusing if some type error unrelated to this library occurs. It would be tough to debug and distinguish which came from Typist and which didn't. For that reason, Typist throws its own custom errors that inherit from `TypeError`, including a helpful message:

{% highlight console %}
TypeError: TypistError: Expected 1,2,3 to be of type String
{% endhighlight %}

## Support

Typist is written in ES5 and should work on all versions of node. Due to dev depenencies, Node v4 or higher is required to run the tests.

[typist]: https://github.com/scttdavs/typist
[flow]: http://flowtype.org/
[dart]: https://www.dartlang.org/
