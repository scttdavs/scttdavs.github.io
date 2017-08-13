---
layout: post
title: "Progressive Rendering with Rails"
photo: prog
meta: post
---

![](/images/prog.jpg)

At my work, we use [Marko][marko] as our templating engine, which supports [progressive rendering][prog_rend] out of the box. So maybe that is why I took it for granted when I decided I would try to implement the same thing for [Explorable Places][ep]. There are a few pages there that are data-heavy and it would help<!--more--> to start rendering the page right away rather than wait for everything to be loaded and rendered.

I expected there to be a gem already for this, in some shape or form, but was pleasantly surprised to find out [Rails supports this out of box][stream], and has since Rails 3! Now, I got this working, but not without a few gotchas I thought I'd share.

## Choose a templating language that supports streaming

This may seem obvious, especially since I mentioned it above, but it wasn't until later I realized I was in a pickle. I chose [Haml][haml] when I first started the project because I liked it and it was a sort of standard in Rails to begin with, with how well it works with Ruby. I never had streaming the rendering of the page in mind back then. As it turns out, Haml does [*not*][no_stream] support streaming. In fact, most templates don't. But [ERB][erb] does, and so does [Slim][slim]. In [benchmarking][template_benchmarks], it seems that Slim is actually _slightly_ faster than ERB, but not enough to make me learn it while getting this up and running, so ERB is what I went with.

Since I only want a few pages to stream for now, I only needed to convert the layouts, templates and partials that those pages touch. This takes significantly less time than converting the whole site. [Haml2erb][haml2erb] helped a lot with speeding this process up.

## Avoid any middleware that alters the html output.

Since we are streaming in chunks, if you have any middleware that alters this, the chunking will be corrupted and invalid and your page won't even load. You'll just get a not-so-helpful error message from your browser. I've read that this includes New Relic's gem, but in my case, there were a few others.

[Bullet][bullet] was easy, but [Rack Mini Profiler][rack_mini] was tougher since it loads automatically and I haven't touched these gems in forever. The mini profiler won't work even if you choose to log to file only. Both of these gems I had to load conditionally behind an environment variable so I could still use them easily from time to time in development.

This may be all you need to get things up and running, but in my case, and hopefully in your case too, there should be one more step.

## Monkey patch to get gzipped chunks working

The [current Rails streaming implementation][stream_imp] works by rendering directly to a chunked body. This means it is chunked, and then gzipped by the `Rack::Deflater` middleware that we use (and you should too!). This corrupts the encoding and again, your page won't load save for a not-so-helpful error message. There is a [good explanation of this on Stack Overflow][so_question] with a link to the Http spec and the example of the monkey patch that I took from.

{% highlight ruby %}
# config/application.rb
config.middleware.use Rack::Chunked
config.middleware.use Rack::Deflater
{% endhighlight %}

The patch expects you to use the `Rack::Chunked` middleware in addition to the Deflater. It overwrites Rails' implementation of streaming so that it only removes the `Content-Length` header and makes sure _not_ to add the `Transfer-Encoding` header. It will rely on the chunked middleware for encoding. It also directly renders the template, rather than to the chunked body, again relying on the middleware to do the chunking. Here is what I used:

{% highlight ruby %}
module GzipStreaming
  def _process_options(options)
    stream = options.delete(:stream)

    super

    if stream
      unless env["HTTP_VERSION"] == "HTTP/1.0"
        headers["Cache-Control"] ||= "no-cache"
        headers.delete("Content-Length")
        options[:stream] = true
      end
    end
  end

  def _render_template(options)
    if options.delete(:stream)
      view_renderer.render_body(view_context, options)
    else
      super
    end
  end
end

module ActionController
  class Base
    include GzipStreaming
  end
end
{% endhighlight %}

The result is that the rack middlewares handle this correctly, finally! Good luck and happy rendering!

[marko]: http://markojs.com
[prog_rend]: https://medium.com/ben-and-dion/progressive-rendering-a-killer-and-under-appreciated-feature-of-the-web-97c789b608c1
[ep]: https://www.explorableplaces.com
[stream]: http://api.rubyonrails.org/classes/ActionController/Streaming.html
[haml]: http://haml.info/
[no_stream]: https://github.com/haml/haml/issues/436
[erb]: https://en.wikipedia.org/wiki/ERuby
[slim]: http://slim-lang.com/
[template_benchmarks]: https://medium.com/@mario_chavez/rails-template-engines-performance-9ba18446895d
[haml2erb]: https://haml2erb.org/
[bullet]: https://github.com/flyerhzm/bullet
[rack_mini]: https://github.com/MiniProfiler/rack-mini-profiler
[so_question]: https://stackoverflow.com/questions/7986150/http-streaming-in-rails-not-working-when-using-rackdeflater
[stream_imp]: https://github.com/rails/rails/blob/v5.1.3/actionpack/lib/action_controller/metal/streaming.rb
