---
layout: post
title: "Composing components with Stimulus JS"
meta: post
---

## How I learned to embrace the untrendy

When Hey email recently launched, I read about their stack (the Hey stack), in depth, and it was the first time I really learned about how they do things at Basecamp, despite using rails for years. I knew about Turbolinks, but I did not know about the Majestic Monolith, or StimulusJS. In a world of React, Vue, Svelte, Stencil, Angular, and countless others, any solution for a SPA that wasn't client-side rendering seemed...boring? unsexy? all of the above?<!--more--> 



## Composing components: Clipboard button and Tooltip

With Stimulus, we have controllers that aptly attach to and control some part of the DOM. This is very flexible and generic, and a controller could control a button or a whole page, made up of several other controllers. This is what I was interested in, and where I think using a library like Stimulus really shines. To demonstrate, I want to show a real world example I came across while adding Stimulus to Explorable Places.

First, I'll start out with a simple tooltip. Our HTML could just look like this:
{% highlight html %}
<div data-controller="tooltip">
  click me
  <span role="tooltip" data-target="tooltip.content">
    some content!
  </span>
</div>
{% endhighlight %}

And our controller could look like:

{% highlight typescript %}
import { Controller } from "stimulus"
import Expander from "makeup-expander"

export default class extends Controller {
  /*
    this controller is for controlling a tooltip
  */

  tooltip: Expander
  contentTarget: HTMLElement

  static targets = ["content"]

  connect() {
    this.tooltip = new Expander(this.element, {
      content: this.contentTarget,
      expandOnClick: true
    })
  }

  disconnect() {
    this.tooltip.destroy()
  }
}
{% endhighlight %}

I'm using a third-party library for simplicity, but also because it best handles accessibility as well. This controller is basically a wrapper that proxies to the underlying Expander. Already, this is super simple and declaritive.

Now, what if another component needs to use a tooltip directly? We could bake that into it, have it import Expander as well, or we can change things slightly so it can use our Tooltip component directly. 

Suppose we need to copy some text, and clicking a button will copy it to the clipboard for us. We'll need a new controller. For this, our HTML with Stimulus declarations could look like:

{% highlight html %}
<span id="importantText">text to copy</span>

<button
  data-controller="copy"
  data-copy-target="#importantText"
  aria-label="click to copy important text">
  <svg class="clipboard-icon">
    <use xlink:href="#clipboard-icon"></use>
  </svg>
</button>
{% endhighlight %}

And our Stimulus controller:

{% highlight typescript %}
import { Controller } from "stimulus"
import Clipboard from "clipboard"

export default class extends Controller {
  /*
    this controller is enabling copying text to the clipboard
  */

  clipboard: Clipboard

  connect() {
    const targetToCopySelector = this.data.get("target")

    this.clipboard = new Clipboard(this.element, {
      target: () => document.querySelector(targetToCopySelector)
    }).on("success", () => {
      // show tooltip?
    }).on("error", () => {
      // do something?
    })
  }

  disconnect() {
    this.clipboard.destroy()
  }
}
{% endhighlight %}

Now we have a button that copies some text, but as you can see, we need to give the user some feedback on what happened. This is where our tooltip comes into play.

We can treat our tooltip element like custom element or web component, and expose some functionality for others to use to directly control it. Let's expose a couple of properties on the tooltip, one that will expand or contract it, and another to set the content. We'll also need a way to disable the default click behavior.

{% highlight typescript %}
import { Controller } from "stimulus"
import Expander from "makeup-expander"

export type Tooltip = HTMLElement & {
  expanded: boolean,
  content: string
}

export default class extends Controller {
  /*
    this controller is for controlling a tooltip
  */

  tooltip: Expander
  contentTarget: HTMLElement
  element: Tooltip

  static targets = ["content"]

  connect() {
    const trigger = this.data.get("trigger")
    this.tooltip = new Expander(this.element, {
      content: this.contentTarget,
      expandOnClick: trigger !== "manual"
    })
  }

  disconnect() {
    this.tooltip.destroy()
  }

  set content(content) {
    this.contentTarget.innerHTML = content || ""
  }

  get expanded() {
    return this.tooltip.expanded
  }

  set expanded(value) {
    this.tooltip.expanded = value
  }
}
{% endhighlight %}

Now, our copy button controller has the ability to target the tooltip, and expand it when it needs to, even setting the content:

{% highlight typescript %}
import { Controller } from "stimulus"
import Clipboard from "clipboard"
import Tooltip from "./tooltip_clipboard"

export default class extends Controller {
  /*
    this controller is enabling copying text to the clipboard
  */

  clipboard: Clipboard
  tooltipTarget: Tooltip

  static targets = ["tooltip"]

  connect() {
    const targetToCopySelector = this.data.get("target")

    this.clipboard = new Clipboard(this.element, {
      target: () => document.querySelector(targetToCopySelector)
    }).on("success", () => {
      this.tooltipTarget.expanded = true
    }).on("error", () => {
      this.tooltipTarget.content = "press Ctrl+C to copy"
      this.tooltipTarget.expanded = true
    })
  }

  disconnect() {
    this.clipboard.destroy()
  }
}
{% endhighlight %}

By designing our components theis way, we can use them as building blocks to construct our pages with, maximizing reuse, and minimizing duplication.
