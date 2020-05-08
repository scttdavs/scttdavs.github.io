---
layout: post
title: "Becoming a Recurser"
photo: recurse.png
meta: post
---

![](/images/recurse.png)

I recently had the priveledge to attend the Recurse Center, which if you have not heard of, you should definitely check out. It was a really interesting time to attend, given the current global pandemic we all are finding ourselves in now. The entire experience was moved online,<!--more--> and done remotely. 

While a lot of the social aspect was lost (but credit is due to the RC team, a lot was retained as well), I was still able to grow and learn about a number of things I had no idea I would tackle. I wanted to cover everything here, as a record of what I learned, and maybe as a starting place for when I can jump back in!

## D3

I had been wanting to learn about [D3][d3] for a long time, and finally I had the opportunity. Infographics, in particular, have been an interest of mind since my graphic design days. In particular, [Edward Tufte][tufte] was someone whose work I always admired. I figured I could build something in D3 in the style of Tufte. After a bit of research, I found [a library][rfancyaxis] written in R that basically accomplished what I was looking for. I used that library as a visual guide to basically build the same thing in javascript.

With that in mind, I was able to publish [fancy-axis][fancyaxis], with its first feature, a rug plot. It implements the same interface as D3 for axes, so that it follows the same pattern and can be used interchangeably. It also adds a couple more methods, since the axes will need to access the chart data now, but it follow the same patterns as the rest of the library, so that should be familiar too. Here is an example:

{% highlight javascript %}
import { rugPlot } from "fancy-axis";
 
// dataset is like [{y: ... }, ...]
const leftAxis = rugPlot.axisLeft(yScale, {
        width: 10,
        color: "rgba(0,0,0,0.1)",
        strokeWidth: 2,
    }).datum(dataset).y(d => d.y);

const bottomAxis = rugPlot.axisBottom(xScale, {
        width: 10,
        color: "rgba(0,0,0,0.1)",
        strokeWidth: 2,
    }).datum(dataset).x(d => d.x);
 
svg.append("g").attr("class", "y axis").call(leftAxis);

svg.append("g")
    .attr("class", "x axis")
    .attr("transform", "translate(0," + height + ")")
    .call(bottomAxis);
{% endhighlight %}

and what that could look like:
<img src="https://github.com/scttdavs/fancy-axis/raw/master/rug-plot-example.png" alt="drawing" style="max-width:400px; display: block;"/>

## Elm

I had wanted to dive into another language, to further broaden my horizons on what are the styles and stardards of another successful and popular language. I at first wanted to try some dialect of Lisp, probably Clojure, but I start also looking at Elm, which is heavily influenced by Haskell.

It is a pure functional language, which I was interest in, and it compiled to Javascript. One thing I didn't know, is that it can also be used to build an entire web application front end, including the UI. In that sense, it can be an entire replacement for libraries such as React, Vue, etc.

I completed the official beginner's tutorial, and built some small components, such as an simple counter. I enjoyed the type system more, but overall I did have trouble getting used to the syntax. I don't think I would pick it up on my own, but I do think I would enjoy working on an existing project that used Elm! Here is a small snippet for clarity:

{% highlight elm %}
import Html exposing (text)

greet : String
greet = "World"

main =
  text (String.join " " ["Hello", greet])
{% endhighlight %}

## WebAssembly

All code eventually runs as machine code. Before it gets to there, it is compiled to assembly, which is a symbolic representation of machine code. Since machine code is specific to the hardware it is run on, so is assembly. So, code must be compiled to something else first, called an intermediate representation. This is more portable now, as it can be run on any computer, and that computer would be responsible for compiling it into its own assembly.

[WebAssembly][webassembly] aims to be another intermediate representation, fitting in right before assembly. It is run in the browser, and the implications are vast. Consider the JVM, which is what Java, Scala, Clojure, and others compile down to. This is considered an intermediate representation. If a compiler exists to compile code written for the JVM into WebAssembly, then any programs written in those languges can now be run in the browser!!

### The end of Javascript?

Not at all, as Javascript is still the language of the web. But WebAssembly can be much faster. Nowadays, Javascript is fast mostly because of the [JIT (just in time) compiler][jit]. This is a special kind of compiler that compiles and optimizes code into machine code on the fly. Due to the dynamic nature of Javascript, it cannot do this ahead of time, it has to do it at run time. But with WebAssembly, since it is compiled much closer to assembly, you cannot retain that dynamism. Types must be known ahead of time, which means the optimization that the JIT does can be done ahead of time, and is more reliable. This is not to say that all apps should be in WebAssembly, but if there is a particular CPU heavy piece of code, it could benefit from being run in WebAssembly.

### So I need to learn Rust now?

Nope, there is good news for us Javascript devs out there. A new language called [AssemblyScript][assemblyscript] has been created and is a subset of [TypeScript][typescript], with all the special types it needs to compile correctly to WebAssembly.

There are some special caveats (with WebAssembly as a whole), such as you can only return numbers. If you return complex type, such as an array, or even a string, the number returned will point to the location in the shared memory buffer where you can retrieve the data. There are some nice loaders though, where this work is done for you, and you can return strings and typed arrays, but nothing else. This is one way in which it ensures that WebAssembly will guarantee quickness by limiting it to those types only.

Here is a sample of AssemblyScript that just adds two 32 bit signed integers:

{% highlight typescript %}
export function add(a: i32, b: i32): i32 {
    return a + b;
}
{% endhighlight %}

## Collaborative text editor

A few years ago at eBay, we typically used [Collabedit][collabedit] for our coding interviews. It wasn't that good, and often we lost connection and had to refresh the page. A coworker and I joked that we should learn how to build one and do it for a Hack Week project one summer. I agreed that it was a good idea, but we never got around to it. However, I decided to dig into it now and see if it could be done!

I found a wonderful [case study][casestudy] that another group made when they tried to solve this same problem. The tool then built is available to try out at [conclave.tech][conclave].

I started building it and successfully got peers to connect to each other over WebRTC via a shared url. Next was the sharing of actual data typed by clients. Central to this feature is avoiding conflicts as everyone types over the same characters. One clever way to solve this is with a new kind of datatype I had not heard of, called a [CRDT or confict-free replicated data type][crdt]. Specifically, I would be dealing with a sequential CRDT.

It works by treating the characters as nodes, and edits to those nodes then become children themselves. Each node keeps track of which user made it. I'm oversimplifying here, but by treating the data this way, it can manage all the edits from various users and ensure that everyone ends up with the same result on their respective computers! This was perhaps the project I had the most fun on, and which I had more time to delve in and get it working like the conclave team did. There is always next time!

As the Recurse Center often says, "never graduate!".

[d3]: https://d3js.org/
[rfancyaxis]: https://www.cl.cam.ac.uk/~sjm217/projects/graphics/
[fancyaxis]: https://www.npmjs.com/package/fancy-axis
[tufte]: https://en.wikipedia.org/wiki/Edward_Tufte
[webassembly]: https://webassembly.org/
[assemblyscript]: https://docs.assemblyscript.org/
[typescript]: https://www.typescriptlang.org/
[jit]: https://en.wikipedia.org/wiki/Just-in-time_compilation
[collabedit]: http://collabedit.com/
[casestudy]: https://conclave-team.github.io/conclave-site/
[conclave]: http://conclave.tech/
[crdt]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
