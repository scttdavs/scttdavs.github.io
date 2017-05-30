---
layout: post
title: "Converting My Site to Jekyll"
photo: jekyll
meta: post
---

![](/images/jekyll.jpg)

## Background (tldr, how I did it next section)

I first built my personal site over 10 years ago. I didn't know much about web development then (even if I thought I did), so at first I stuck to what I knew most, flash. Those were the days, an entirely animated site. It may have been hell to maintain, and even to use sometimes, but they sure were fun to build.

After flash went out of style, very quickly, I just wanted a simple site but one where I could add content without having to add <!--more--> a new static page. I didn't know rails or node then, so making an app was pretty much out of the question. So I turned to what was in favor then and still is now, Wordpress. But Wordpress had its own problems. It wasn't very enjoyable to work in for one. The dashboard was messy, plugins seemed like an easy solution, but in reality were a mess. Often they conflicted with each other, or you had to search around to find the best one that actually works for what you're trying to do. I defintely felt tied up and limited to what I could do with it.

Leaving Wordpress, I then switched to an actual CMS, one I'd heard was good: [CouchCMS][couch]. Couch was heaven compared to Wordpress. Simple to set up, drop the tags in your HTML files, no need to convert to Wordpress' forgetful and complicated template system. And again, this was fine for a while, but I would still forget the log in portal url, my password, it's still dealing with php, and at the end of the day, I'm still having to ftp all my changes whenever I want to add a part of my site (although [this][gitftp-post] was pretty helpful). The kicker was when I got a security update that I needed to patch from them or else I would be at risk of being hacked. It was, yet again, time for a change.

## Enter Jekyll

Now I've known about Jekyll for a few years, but by then, my site was up and running fine and I didn't have a desire to switch then. My how times have changed. I dived right in and within a day I had my site essentially done. Even better with Jekyll, you can get free hosting with Github Pages (since that's what they run on and Github created Jekyll). So with my site now running Jekyll, I get:

- Free hosting
- custom domain names easily added
- no more ftp, just push changes to repo and watch them go live
- free backups, it's all on github after all
- no dbs, even more portable should I decide to switch hosts (for some reason)
- secure, no need to log in and keep passwords, nothing to hack into or get because nothing sensitive is there

I'll go over the basic steps of how I did it below as a feel a quick introduction helps a lot before turning to Jekyll's (really good) [documentation][docs] to learn more and deep dive into specific features.

## What is Jekyll?

[Jekyll][jekyll] is a static site generator. That means no logging in, no databases, just files sitting on your server and people viewing them. With Jekyll, you create templates for your site, add content via separate files (one file for each post, for example), and jekyll processes the whole thing, pumping out all the static files you need for your site. So with a lot of code reuse, we have just a few templates, our content, and then a whole lot of output. 

## Installing Jekyll

This may be the most technical part of the process, just getting the necessary tools installed. But first, you need to install Jekyll, so in terminal:

{% highlight console %}
gem install jekyll
{% endhighlight %}

And then create your project wherever you want it:

{% highlight console %}
jekyll new kickass-site
{% endhighlight %}

and then change directories to your site and run your server. You don't need to start your server *on* your server to have your site be viewable, but doing this locally is what generates all your files for you.

{% highlight console %}
cd kickass-site
jekyll serve
jekyll new my-awesome-site
{% endhighlight %}

You can now view your site at `http://localhost:4000` Open up your kickass site in your favorite editor and get to work!

## Jekyll File Structure

When you open up your site you'll see a lot is created for you. I'll go over most of them here to explain. 

Folders starting with `_` are special. They are used by Jekyll for specific purposes. For example `_posts` is where all your posts live (not rocket science here). `_includes` are where partials live. So you can do:

{% highlight liquid %}
{% raw %}
{% include footer.html %}
{% endraw %}
{% endhighlight %}

in your template to include a footer you made. By default, Jekyll will look in the `_includes` folder. `_layouts` are where your layouts live, which probably all of your pages will have a layout associated with them. `_site` is where all your converted files for your static site live. This should never be used for anything else. If you upload your site to your own host, these are the contents that you upload.
I think the rest are easily understood. There is a `_drafts` folder you can create yourself for drafing posts. When you run your server with a `--drafts` flag, it will convert these for you for previewing them.

The `css` folder comes with a sass file for you. At the top it explains that you need front matter there, but not in any files in the `_sass` folder. I'll explain front matter below, but all files in the `_sass` folder will be converted automatically. Jekyll has sass support out of the box now and you can add coffeescript support with a gem. See [here][assets] for more about Jekyll assets.

## Templates and Layouts and... Front Matter?

Jekyll uses [liquid templating][liquid] developed by Shopify. It supports all of the standard [tags][liquid-tags] and [filters][liquid-filters] and even a [few more][jekyll-filters]. So if you have any syntax questions revolving around that, you know where to look. But Liquid is extremely simple and similar to a lot of other templating languages, like mustache and handlebars.

Also what is very important is [Front Matter][frontmatter]. Front Matter is the meta information denoted in YAML at the top of every page and post (or any other content you added). It includes basics such as title, which layout to use with it, and any other information you want passed along up the chain that is usefor for your site. Here's a sample from my site:

{% highlight yaml %}
layout: post
title: "Caching model associations in rails"
meta: post
{% endhighlight %}

Any of these variables are available with the page variable, so `page.title` for instance in my example above, would return that value. Any variable associated with your site (and set in the _config.yml file) are available on the site variable, like: `site.url`. More on variables [here][variables].

You can have a look at my finished site repo [here][my-site] for reference and how this may all tie into each other. There is a quite a bit that will probably be involved in making your templates and layouts for your site, but that's not what this post is about. So I'll leave that to you. But once that is done, time to put that shit up on the web!

## Github hosting

Sure, you can pay for your own hosting, but why bother when you can get unlimited free hosting that is super fast? Just upload to github, it's ridiculously easy. First thing's first, create a new repo on github that has this naming scheme: `username.github.io` where "username" is *your* github username. It must be named this way exactly. 

Second, add `_sites` and anything else you don't want on your public repo in your .gitignore file. Github will generate your site for you, so no need to commit the `_site` folder. Also, maybe you have user/admin info stored in `_data` as [data files][data-files], so you may want to exclude that too.

Third, just push to your repo. That's it. You should be able to visit your site within a couple seconds at *username*.github.io. Woohoo!!! 

## Custom Domain Names

[Support][custom-domain] for this is easy. First just creat a new file in the root of your project called `CNAME` and on the first line, put your domain name. For mine, I just put "sco.ttdavis.com". Commit to your repo. Next, you'll need to update your DNS settings for your domain. For a apex domain, you'll need to add a new A name. And for subdomains, add a new CNAME. For more info on your case, see here.

## That's it!

Feel the sun shining on your face? The nice cool breeze? The ease of deploying to your freely hosted site in seconds? Feels great to be alive doesn't it?


[jekyll]: http://jekyllrb.com/
[jekyll-filters]: http://jekyllrb.com/docs/templates/
[frontmatter]: http://jekyllrb.com/docs/frontmatter/
[variables]: http://jekyllrb.com/docs/variables/
[data-files]: http://jekyllrb.com/docs/datafiles/
[liquid-tags]: https://github.com/Shopify/liquid/wiki/Liquid-for-Designers#tags
[liquid-filters]: https://github.com/Shopify/liquid/wiki/Liquid-for-Designers#standard-filters
[docs]: http://jekyllrb.com/docs/home/
[assets]: http://jekyllrb.com/docs/assets/
[liquid]: https://docs.shopify.com/themes/liquid-documentation/basics
[couch]: http://www.couchcms.com/
[custom-domain]: https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/
[my-site]: https://github.com/scttdavs/scttdavs.github.io
[gitftp-post]: {% post_url 2014-05-08-git-ftp %}