---
layout: post
title:  "Automating Jekyll static website delivery with Jenkins"
date:   2016-01-15 00:37:00
categories: jenkins jekyll static website delivery ci continuous integration
tags: [jenkins, jekyll, ci, continuos-integration, rvm, gem, git, gitlab]
---

So, i've decided i needed a way to automate my static blog publishing and decided to go with Jenkins to do all the heavy lifting for me, 
i could have gone the easier way by using hooks or simple bash scripts but i've decided i need to practice my Jenkins skills a bit.

My Jenkins instance is on the same machine as my webserver so deployment was real easy with just building and copying built site to my web server's folder.

While I am writing this I am using `--watch` option on Jekyll to check my formatting on a localhost auto-refreshed instance.

{% highlight tcsh %}
$ bundle exec jekyll serve --watch 
{% endhighlight %}

This way I can track the changes and get the "wysiwyg" feeling for my post.

Once I am ready to publish the post I just commit my changes to my GitLab instance and then Jenkins steps in to build the site and deploy it to my webserver.
Basically what Jenkins has to do is install all the gems with ``bundle install``, build website to ``_site`` folder and copy the files, all real easy.

In order to do gem bundle install for website template i use i needed to install RVM - followed instructions from [RVM install tutorial](https://rvm.io/rvm/install)

For future reference:

{% highlight tcsh %}
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
$ \curl -sSL https://get.rvm.io | bash
{% endhighlight %}

Also I needed to add Jenkins to rvm group so it can install gems:

{% highlight tcsh %}
$ sudo adduser jenkins rvm
$ sudo service jenkins restart
{% endhighlight %}

After having everything set up I created a new `Freestyle project` in Jenkins, set Git as a SCM with my website repo, enabled GitLab plugin to `Build when a change is pushed to Gitlab` and made 
sure that `Build on Push Events` was true. In order for GitLab to notify Jenkins of new pushes to repo you need to setup Web Hook for project in GitLab.
 
I've set Execute shell with following "shell" script:

{% highlight tcsh %}
rvm use "ruby@gemset"

set -e

cd jekyll-uno/
bundle install
bundle exec jekyll build
rsync -av --delete ../_site/* /var/www/website/
{% endhighlight %}

And that's it. 