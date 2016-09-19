---
layout: post
title: "The What, Why, and How of HTTPS"
photo: ssl
meta: post
---

![](/images/ssl.png)

In the past, you would probably only really check if a website used https if you are purchasing something, or *maybe* if you were logging in. In the last few years, there has (rightly) been a large movement to better ensure privacy and security. Why not have every site and every page be encrypted? That's a good question, and one Google has already answered for itself. Not only will Google start [shaming websites that are not https][shaming], they will also [give an SEO boost][seo] to those that are. <!--more-->If your customer's peace of mind wasn't enough, you now have more than enough incentive to make the switch.

I recently decided to do this for [Explorable Places][ep], for the reasons above and also in preparation of supporting Stripe and credit card payments. While it is simple to make this change, there is a few things worth learning (and documenting) and some snags you may run into.

## What is SSL/HTTPS?

Otherwise know as SSL (secure socket layer), HTTPS is a protocol, in the same realm as regular ole HTTP or FTP, that is encrypted and was developed by Netscape. It listens on port 443 as opposed to port 80. You'll need just a couple things to enable it and start using in on your site: You'll need a **_certificate_** file and a **_private key_** file.

Without going into too much detail, the certificate is signed by an authority that the browser trusts (it only trusts known certificate authorities). I'll get into signing in a bit though. In addition to being signed, the certificate **also contains a public key** that will be sent to the browsers. When you make a request to the server from your browser, it will encrypt that data with that public key. The data in that request can only be decrypted with the private key residing on the server.

So the server decrypts the request with its private key and does whatever it needs to with it. It'll then return a response that it encrypts with said key again. This response in turn can only be decrypted with the public key. It's actually quite nice how they work together.

Now it's important to point out that the data being sent to the browser is encrypted, but that doesn't mean it's secure since all it takes to decrypt it is the public key, which anyone has access to. But all data sent *to* it, that is locked down. &#128274;

Back to signing the certificate. You could create your own certificate and sign it yourself (as opposed to a known authority) and use that. Many people do this when developing locally or for apps within a private network. But to public facing apps, self-signed certificates can't be trusted since nobody knows you and you could be up to no good. Browsers will throw up a very scary looking warning page if you sign your own certificate, and basically freak the hell out of anyone visiting your site.

So you'll need to purchase one, which will run from very cheap with no paperwork (insures for very little, good for personal sites), to very expensive and lots of paperwork (insures for a lot). The latter requires you to prove your business is who they say they are, and can allow you to include that name next to the URL in the browser. This also looks very nice, so it may be worth it to you if security is particularly a big deal.

I went with just the plain old domain verification option, which can be issued immediately with no paperwork, I just have to prove that I own the domain. This can be done with an email that is associated with the domain, easy peasy.

## How do I get one

Once you've decided on what you want and purchased it, you'll need to request the certificate from the issuing company. To do this, you'll need a certificate signing request (CSR) file. This can be created with many tools available around the internet, most of which cost money. Or you could do it on the command line with openSSL:

{% highlight sh %}
openssl req -new -newkey rsa:2048 -nodes -keyout YOUR-DOMAIN.key -out YOUR-DOMAIN.csr
{% endhighlight %}

This will create a token with 2048 bit encryption, and output your CSR and your private KEY file which you will need for later. When you enter this command, it will prompt you through a set of questions, the only one worth noting is the `name` field, which should be your domain name (if you are doing a domain verification certificate like me).

When finished, submit this CSR to the issuing company and answer whatever other questions they have (it's very easy), and you will get issued your certificate (CRT) file within a few minutes. You should now have your certificate `.crt` and private key `.key` files.

## Intermediate Certificate

Depending on the type of certificate you purchase, you may be required to include an intermediate certificate. An intermediate certificate lies between the domain certificate (yours) and the root certificate of the issuing authority. This is done to ensure its authenticity and maintain the "Chain of Trust"&trade; because it's insecure to include the root certificate itself. To include this, you will need to chain them together. Chaining is nothing more than file concatenation.

You will be able to just download an intermediate certificate from the issuer as there is no customization or CSR needed for them, everyone gets the same one. Then either in a text editor (with no formatting) or through the command line, concatenate the files with your domain certificate first and intermediate second.

## Telling your server to use them

Now that you are all set, you'll need to get your server configured. This includes uploading the chained certificate (or regular domain certificate if you did not need to chain) and the KEY file from the beginning. You can put these anywhere. I have a linux server and put the certificate in `/etc/ssl/certs` and the key in `/etc/ssl/private`.

Then in your nginx configuration file (or where ever your server config is), you will need to tell it where those files are. You will also need to tell the server to listen on port 443. For nginx and unicorn this will look something like:

{% highlight nginx %}
server {
    listen 443;
    ssl on;
    ssl_certificate /etc/ssl/certs/YOUR-DOMAIN.crt;
    ssl_certificate_key /etc/ssl/private/YOUR-DOMAIN.key;

    # EVERYTHING ELSE IS THE SAME AS PORT 80 CONFIG EXCEPT:

    location @unicorn {
        # YOU'LL WANT TO ADD THIS TOO IF YOU USE RAILS
        proxy_set_header X-Forwarded-Proto https;

        # OTHER SETTINGS ARE THE SAME
    }
}
{% endhighlight %}

If you use rails, you'll likely use `force_ssl` in your controllers. I use it in the application controller to force https for all routes. It relies on the `X-Forwarded-Proto` header to work and is why rails needs it.

Then just restart your server, for nginx: `sudo /etc/init.d/nginx restart`

Now when you visit your site with `https://`, it should work and you should see the little green lock. If not, you may want to troubleshoot with the issuing company to get it resolved. Also, you will need to make sure **ALL** assets you serve from the page (js, css, images, fonts, etc) are served via SSL as well, or the browser will warn the user, or possibly not work at all.

Welcome to the secure side! &#128272;

Now I just need to wait for Github to allow for https of github pages with a custom domain and I can revisit this article.

[shaming]: http://motherboard.vice.com/read/google-will-soon-shame-all-websites-that-are-unencrypted-chrome-https
[seo]: https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html
[ep]: https://www.explorableplaces.com
