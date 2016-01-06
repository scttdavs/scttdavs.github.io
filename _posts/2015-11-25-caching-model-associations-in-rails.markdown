---
layout: post
title:  "Caching model associations in rails"
date:   2015-11-25 19:02:31 -0500
---

I've been working on a project recently called [www.explorableplaces.com][exp], and I added memcached caching with [Dalli][dalli]. So I'm basically trying to cache anything and everything that can help our performance. 

Normally, you will see guides and articles show you caching with an example like this:

{% highlight ruby %}
def cached_images
  Rails.cache.fetch(['Articles', id, 'images', updated_at.to_i]) do
    images.to_a
  end
end
{% endhighlight %}

And while this now works, if I eager load an association, it will not <!--more--> call my "images_cached" method but the default one instead, so no gain there.

When I was thinking about this more and more, a lightbulb went off in my head. Instead, you can do something like this:

{% highlight ruby %}
class Article < ActiveRecord::Base
  has_many: :images
  has_many: :cached_images, class_name: "Image"

  def cached_images
    Rails.cache.fetch(['Articles', id, 'images', updated_at.to_i]) do
      super.to_a
    end
  end
end
{% endhighlight %}

## So What's Going On Here?

This adds a new association that basically just returns all an article's images. So why did I create a new one? Well we still need our controllers to work with creating new images like "@article.images.new". Also, the advantage here is now I can eager load images: "Article.all.includes(:cached_images)" which will return all cached images with each article.

When it's run, rails tries to fetch it from the cache using a key created from the array of values passed into it. If it finds something, it returns it. And if not, it runs the code in the block and returns that (which is then cached for future use). In that block, super just calls the original "images" method to make the call to the db and get those images.

## Updated_at? What Are These Keys?

Why do I have "updated_at" in there? Basically, once the model is updated, this would mean the key would be updated, and essentially expire the old cached value, fetching a new one. This is a safer feature so your data doesn't get stale. For our example, if the article has an id of 5, then our key would look something like "Articles/5/images/1448145415".

You may have wondered, what if I update an image? Then the cache will not be expired but I will still get old data? We can easily fix that in our images model:

{% highlight ruby %}
class Image < ActiveRecord::Base
  belongs_to: :article, touch: true
end
{% endhighlight %}

With "touch: true", whenever the image is created, updated or destroyed, it will touch the article it belongs to, updating its "updated_at" time. Sweet!

## And To_a?

And you may also be wondering why ".to_a". Well, without that, it will actually just cache the ActiveRecord association and not the values. That's bad, and would just throw an error on us if we try to retrieve it. Calling "to_a" on it forces that association to actually make the call and get our data, which is returned as an array so we can cache it.

That's it, hope you found this as useful as I did!

[exp]: http://www.explorableplaces.com
[dalli]: https://github.com/mperham/dalli
