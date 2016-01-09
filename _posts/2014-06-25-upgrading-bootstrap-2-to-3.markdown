---
layout: post
title:  "Upgrading Bootstrap 2.x to 3"
meta: post
---

![](/images/bootstrap.jpg)

When we started building [EastMeetEast][eme] last August, Bootstrap 3 was still a release candidate. It didn't seem to make very much sense to implement it on our production site when it may have bugs or be updated and changed frequently. 

After just a few weeks of work, the official release of 3.0 came out. At that point, we were more concerned with getting the product up and out than now backtracking to <!--more--> upgrade to 3. Well the time has come, now 9 months later when things are much more quiet, to tackle this sucker.

I'm writing this article as I come across common "gotchas" when migrating. For a detailed list of all the changes, you can [view them here][docs].

## New Grid

This is probably the most obvious, but there's a new grid. It's pretty simple but is based on "mobile first" design.

{% highlight html %}
<div class="span4 offset2"></div>

//now becomes

<div class="col-sm-4 col-md-offset-2"></div>
{% endhighlight %}

The prefixes sm, md, lg, all refer to the screensizes that the container will be collapsed in. So you can have more than one grid class per element, a "col--4 col-lg-2" or something. There also is no more "row-fluid" class. There is only the "row" class and will always be fluid.

## Helper Classes

A lot of these have also changed or added new ones. These include:

{% highlight html %}
<label class="sr-only"></label>  // screen reader only, will hide from user's view

<div class="hidden-xs"></div> // will hide on mobile. Follows same prefix convention as grid classes

<div class="alert danger-alert"></div> // error-alert is deprecated
{% endhighlight %}

## Navbar

This has changed a lot and is probably better just to [read the docs][docs]. The largest difference, though, is there is no more "navbar-inner".

## Buttons

The buttons have simplefied A LOT. They are now flat which makes them super easy to customize. With them, you can edit your buttons with "btn-danger, btn-info, btn-warning, etc" classes, update your Sass/Less variables like "$brandcolor-primary", or created a custom class with a mixin:

{% highlight sass %}
.btn-brown {

  @include btn-variant(white, brown, brown);

}
{% endhighlight %}

Note: all references to "error" are now "danger", hence the new class "btn-danger" (and also "alert-danger")

## Responsive Images

Responsive images are not responsive out of the box anymore. You'll need to add a class of "img-responsive" to any image you want to behave that way. You may be able to get around it by just adding

{% highlight sass %}
img { max-width: 100% }

// or

img { @extend .img-responsive }
{% endhighlight %}

but it may differ case to case, so use your discretion. For instance, in rails, you may have an image helper like "image_tag(256)" that spits out an image with width and height set to 256px. Using max-width alone on this will cause some problems, so it's best to use the class name. Alternatively, you may have some images styled in such a way that "img-responsive" will break their layout by setting the height to "auto". Just be aware and careful with these.

## Forms

If you use bootstraps form styles, they have changed a lot. "Control-group" is now "form-group", "input-block-level" is now "form-control". "Controls" and "controls-row" have been removed with no replacement. Have a look at the [details][details] for more specific info. If you have any plugins, gems or other third party software that deals with forms, then you will have to check and see if they need to be updated as well. One such popular one is...

## Simple_form With Rails

If you use Rails, and you use the very popular gem simple_form, then you are probably wondering why your forms looks like shit in bootstrap 3. Well, that's because you need to update your gem. An official release is not out yet as of this posting, but there is a release candidate that you can use. For more info, [check this out][check].

Those were all the major steps in upgrading our site to Bootstrap 3. It involved some other little adjustments here and there, but those were specific to our site and not as big as these. Good luck!

[eme]: http://eastmeeteast.com/
[docs]: http://getbootstrap.com/components/#navbar
[details]: http://getbootstrap.com/migration/
[check]: http://blog.plataformatec.com.br/2014/04/bootstrap-3-support-for-simple-form/