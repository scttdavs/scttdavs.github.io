---
layout: post
title: "HTTPS with Github Pages (and Custom Domains)"
photo: ssl-ghp
meta: post
---

![](/images/ssl-ghp.jpg)

[I've posted before]({% post_url 2016-01-26-converting-my-site-to-jekyll %}) about how I converted this site to Jekyll and hosted it on Github Pages. I'm a huge fan of this and it was really exciting last year when they announced support for https with Github pages. But alas, this is for the `github.io` urls, not custom domains.<!--more--> I'm to the point now where I do not want to wait any longer if there are other solutions out there. I would like to avoid the [impending Chrome death mark](https://motherboard.vice.com/en_us/article/google-will-soon-shame-all-websites-that-are-unencrypted-chrome-https), as well as start testing out and using service workers with this site. As it turns out, there is a pretty simple way of doing this using Cloudflare.

## What is Cloudflare

In a nutshell, it's a service for a proxy that lies between origin (your server) and the user. This allows you to serve cached content from their edge CDN servers all over the world, mitigate DDOS attacks, and add SSL support, among many other things.

## How?

There are plenty of tutorials out there on how to do it, [here's a great one here](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/). But if you already have github pages set up, then it just comes down to a few steps:

1. Create a Cloudflare account or log in
1. Create a new site (even if you did this a long time ago, the new creation steps may have changed)
1. It will walk you through and scan your DNS settings. Make sure your A or CNAME settings are correct in Cloudflare once it's done.
1. Change your nameservers of your domain to match the ones Cloudflare gives you.
1. Change your SSL settings for your site to use "Full".
1. Set a page rule to redirect all traffic to https. I used `http://*ttdavis.com/*` to `Always use HTTPS`.

And that's it! It will most likely take a few hours to update, which sucks because your site will be down in the mean time. But at least it's simple and it's free! Now with Cloudflare, Jekyll, and Github Pages, you get free hosting and free SSL certs. 🍻
