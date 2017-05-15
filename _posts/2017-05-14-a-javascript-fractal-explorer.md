---
layout: post
title: "A Javascript Fractal Explorer"
photo: mandelbrot2
meta: post
---

[![](/images/mandelbrot.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=200&er=5&co=true&j=true&ci=0.15&cr=-0.79&zi=0&zr=0&i=0.00366412213740458){:target="\_blank"}

A funny story about me is how I got into web development. It essentially boiled down to me learning, and then becoming obsessed with, [making programs on my TI-83 Plus calculator][ticalc]{:target="\_blank"} in a BASIC-like language designed just for the calculator. This made it a perfect portable IDE for someone learning how to program. I made [minesweeper](http://www.ticalc.org/archives/files/fileinfo/288/28883.html){:target="\_blank"}, [lights out](http://www.ticalc.org/archives/files/fileinfo/295/29524.html){:target="\_blank"}, [D-Star (with custom level editor, I might add)](http://www.ticalc.org/archives/files/fileinfo/296/29633.html){:target="\_blank"}, and a program to view fractals! Well, apparently I forgot<!--more--> to upload that last one. And so began my obsession as an adult to relearn and recreate this with my new fancy web development skills I've picked up in the last 15 years. So basically, [I did it in Javascript](http://sco.ttdavis.com/mandelbrot/){:target="\_blank"}.

After some quick digging, I realized I was way smarter as a high schooler than I am now. How did I do this before? Complex numbers, complex plane, how to iterate correctly and determine if it has escaped toward infinity. And after I figured all that stuff out, I had to tackle more problems, how to zoom in, how to colorize (both grayscale and color as the calculator had neither). This was going to be a tough task.

(Also I'd like to point out all images in this post link to the fractal viewer itself, to see that image live)

## No Framework

I wanted to **not** use a framework, as I did not see the need. This was a single page, single page application. 99% of the work would be done drawing to the canvas, and none of the inputs were related or reliant on any other elements. But, I did take a reactive approach, and it was cool to implement my own stateful app that has a `setState` function and batch renders state changes asynchronously. This part was definitely a little tricky but fun to work on.

## There's No i in Javascript

There's no I in team and the same is true for Javascript. No I'm not talking about _that_ "i" because I see it there after the r and before the p. I'm talking about [imaginary numbers](https://en.wikipedia.org/wiki/Imaginary_number){:target="\_blank"}. When I made this on my graphing calculator, of course there is _i_, and I used it. It made calculating the equation easy. But with javascript it became a bit more difficult. Luckily, math in general tends to have lots of shortcuts, and that was also the case here. Here's the magic for calculating Z<sub>new</sub> = Z<sub>old</sub><sup>2</sup> + C where both Zs and C are complex numbers:

<div class="pre-wrap">
{% highlight javascript %}
ZNewReal = Math.pow(ZOldReal, 2) - Math.pow(ZOldImaginary, 2) + Cr;
ZNewImaginary = 2 * ZOldReal * ZOldImaginary + CImaginary;

absoluteValue = Math.sqrt(Math.pow(ZNewReal, 2) + Math.pow(ZNewImaginary, 2));
{% endhighlight %}
</div>

With the absense of i also comes an alternative way of dealing with complex numbers. We must refer to both the _real_ and _imaginary_ parts separately (where a complex number is written `x + yi`, x being the real part and i being the imaginary part).

[![](/images/mandelbrot2.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=450&er=5&co=true&j=false&ci=0.15&cr=-0.79&zi=0.24107896893625036&zr=-0.7312281391638837&i=0.000005938435328825743){:target="\_blank"}

## Zooming In

Zooming in was not, in and of itself, that difficult. When the user clicks and drags a box to zoom in on, just save the left, right, top and bottom boundaries and that's that...right? Well, as soon as you open that link in a window with a different aspect ratio, it gets all mucked up. Instead of saving the "viewport window", I needed to save the origin (middle coordinate of the screen), and the delta of values between the x plane (real numbers) and y plane (imaginary numbers). This way I could render and view, at any zoom level, on any device and it will be centered and look correct.

## Smoothing Color

When rendering, I would get pretty noticable lines where values would change abruptly. While this was mathematically correct, I've seen many more examples on the internet that looks much smoother. Since I am a very visual person (why I love working on the front end, and hey, I even have a BFA), this smoother kind of look is what I wanted to go for. A pretty clever way to smooth color, [I found here](http://www.karlsims.com/julia.html){:target="\_blank"}. That was way easier than I thought it would be!

<div class="pre-wrap">
{% highlight javascript %}
const newNumberOfIterations = actualNumberOfIterations - Math.log2(Math.log2(finalAbsoluteValueOfLastIterationResult));
{% endhighlight %}
</div>

Sweet variable names, no?

## The End

All in all, this was a lot of fun to work on and hope others enjoy it at least a fraction as much as I did. [Check it out here](http://sco.ttdavis.com/mandelbrot/){:target="\_blank"}, or just look at my screenshots below to see what all this talk is about (remember they are clickable!).

[![](/images/mandelbrot3.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=250&er=5&co=true&j=true&ci=0.15&cr=-0.79&zi=0.4728578244274809&zr=-0.20152671755725204&i=0.0004637404580152672){:target="\_blank"}
[![](/images/mandelbrot4.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=150&er=5&co=true&j=true&ci=0.008&cr=0.28&zi=0&zr=0&i=0.00366412213740458){:target="\_blank"}
[![](/images/mandelbrot5.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=50&er=5&co=false&j=true&ci=0&cr=-1.476&zi=0&zr=0&i=0.00366412213740458){:target="\_blank"}
[![](/images/mandelbrot6.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=55&er=5&co=true&j=true&ci=-0.01&cr=0.3&zi=0&zr=0&i=0.00366412213740458){:target="\_blank"}
[![](/images/mandelbrot7.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=50&er=5&co=false&j=true&ci=1.04&cr=-0.162&zi=0&zr=0&i=0.00366412213740458){:target="\_blank"}
[![](/images/mandelbrot8.jpg)](http://sco.ttdavis.com/mandelbrot/?mi=200&er=5&co=false&j=true&ci=0.3&cr=-0.7&zi=0&zr=0&i=0.00366412213740458){:target="\_blank"}

[ticalc]: http://www.ticalc.org/archives/files/authors/62/6241.html
