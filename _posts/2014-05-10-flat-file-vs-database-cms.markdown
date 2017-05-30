---
layout: post
title: "Flat File vs Database CMS"
meta: post
---

## What Is A Flat File CMS?

…Says just about everyone… “It is kind of cool sounding.” “Is it related to flat design?” “Everything seems to be going flat these days.”

No, it’s not related at all to any flat design or flat anything else. A flat file CMS means that there is no database. Instead of retrieving content for a single page from a database, an actual html/php/text file is created with that content in <!--more--> it. You could find it and open it up in a text editor and it would have all (or almost all) of the stuff you would see when you view it on the web.

If you tried this with a traditional database CMS, well, there would be no file there. There would be a single template file with variables for where the content goes:

{% highlight html %}
<:cms show variable_foo_bar />
{% endhighlight %}

It grabs the values of those variables from a database and plugs them in before delivering the generated html/php file to your browser.

## Why Is This Better, If At All?

It does seem like taking a step back in terms of technology, and it is actually, but it offers a couple advantages.

No database means you don’t have to fool with a database. Need to move your site? Drag and drop that sucka. No more having to migrate the database, which can be a headache to some.
Security. Traditionally, hackers will go after databases first. If there is no database, there’s nothing to get at.
Your site is now 100% version controllable. This one may be their biggest selling point. You can use git to add version control to your site. You can do this with any site, but it gets pretty tricky with databases and most don’t do it. This way, your local copy is the same as your controlled copy (on github for example) which is the same as your live copy. You can push or pull changes to any of these. Easy peasy.
Speed. This is especially true when comparing to wordpress, which is overkill for just about all websites that use it. But if you don’t have to query a database, generate then content, then deliver it, the site tends to load a lot faster.

## Well, Is It Worse In Any Way?

It can’t all be sunshine and rainbows with a flat file CMS, so there must be some downsides.. And there are. Most CMSs that are flat file don’t allow any dynamic content (basically only have HTML/CSS/JS). Statamic is one that does however, as add-ons which cost extra.

Databases are amazing for searching. Searching with a flat file CMS (if they even feature it) may not be up to snuff with a database, especially for large sites with thousands of files.

They can get bloated. Again, for large site, this means thousands of html/php/text files.

Version controllable is not important to the client. Pretty much any client will log on to the cms from a browser to make changes to their site. They will not make a local version, push it to github, and push it to the server. Very quickly all repositories will be out of sync and create a headache for any developer looking to keep them all the same.

Maybe someone more code inclined could write a script to keep them in sync when a change is made (I have no idea), but if you are going to do that and are that technically savvy, why not just use a database and back it up?

## Which Should I Use?

Overall, I would stick with the database driven CMS. Other than maybe wordpress, working with a database is easy and not scary once you do it. Your site will be faster and more rich in available features. It will scale well and can work on large and small sites alike.

I would consider using a flat file cms for a personal site or blog (where the developer/designers themselves are in total control) or small sites and static sites. There won’t be much drop off at all with a small site, so it can and does work well for lots of people and may be the perfect fit for you too.

Here is a list of popular flat file cms’s:

[Getkirby.com](Getkirby.com) free  
[Statamic.com](Statamic.com)  
[Pico.dev7studios.com](Pico.dev7studios.com) free  
[Jekyllrb.com](Jekyllrb.com) free (technically not cms)  
[Siteleaf.com](Siteleaf.com) (static sites only)  

and a list of great non wordpress cms’s:

[Grabaperch.com](Grabaperch.com)  
[Surrealcms.com](Surrealcms.com) free trial  
[CouchCMS.com](CouchCMS.com) free (pay to whitelabel)  
[AnchorCMS.com](AnchorCMS.com) free  
[Builtwithcraft.com](Builtwithcraft.com) free for simple sites  
[Habariproject.org](Habariproject.org) free  