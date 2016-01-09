---
layout: post
title: "Uing PG (postgres) on Mavericks"
meta: post
---

Heroku only uses Postres for databases. That's fine, so you just need install it locally for developement. Well, you are probably pulling your hair out trying to install the pg gem on Mavericks. That's because you can't (as of yet anyway). The good news is there's a quick and easy way.

Just install the [Postgres App][pg], then install the gem in terminal, pointing it to the new config file.

{% highlight ruby %}
gem install pg -- --with-pg-config=/Applications/Postgres93.app/Contents/MacOS/bin/pg_config
{% endhighlight %}

Now, make <!--more--> sure the config is there first. If you are using another version of the app, you may have a different name there for "Postres93.app". But that should do it. Run "bundle install" and click the app to get postres running. Your rails project should work fine now.

[pg]: http://postgresapp.com/