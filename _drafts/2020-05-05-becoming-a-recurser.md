---
layout: post
title: "Becoming a Recurser"
photo: ssl-ghp
meta: post
---

![](/images/ssl-ghp.jpg)

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

[d3]: https://d3js.org/
[rfancyaxis]: https://www.cl.cam.ac.uk/~sjm217/projects/graphics/
[fancyaxis]: https://www.npmjs.com/package/fancy-axis
[tufte]: https://en.wikipedia.org/wiki/Edward_Tufte

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
