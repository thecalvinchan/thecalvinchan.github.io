---
title: A Ghostly Upgrade
layout: post
permalink: a-ghostly-upgrade
published: true
---
A week ago, I published a blog post on how [learning to code isn't a long term solution to revitalizing America's economy](http://thecalvinchan.com/blog/2014/03/04/tech-innovation-part-1/). It was part one of a three part series that focuses on fixing a "broken" education system. Don't worry, I haven't forgotten about parts two and three. I just couldn't bring myself to continue publishing on *Wordpress*. It was slow and clunky, and I was tired of fighting with its difficult text editor. I was ready for something modern and sleek – I wanted a fast, lightweight blogging platform. Then I chanced upon [*Ghost*](https://ghost.org/).

*Ghost* caught my interest because it runs on *Node.JS* (I'm a huge *JS* fan). When I found out that it supports editing in *Markdown*, I was sold. With finals week coming up and nothing better to do ~*sarcasm*, I decided to ditch *Wordpress* for *Ghost*. After all, how hard could it be? As it turned out, much harder than it seemed.

##The SEO problem

I've been running my blog off *Wordpress* for the last two years. In those two years, it has experienced enough traffic for SEO to be a significant factor in the process of migrating to *Ghost*. I wanted to import my old blog posts into *Ghost* with their original slugs (urls) to preserve their *Facebook* likes/shares, *Twitter* retweets, and SEO rankings. This seemed simple enough. After importing the posts, I would just modify the slugs to match what they were before. My old *Wordpress* blog served posts with the following slug format:

	http://thecalvinchan.com/blog/%year/%month/%date/%title/
    
The problem is that *Ghost* doesn't currently support slugs with forward-slashes (/). Therefore, when I imported my posts, the slashes were stripped and the slugs became:

	http://thecalvinchan.com/blog/%year%month%date%title
    
*Ghost* slugs cannot have forward-slashes. At least, not as of right now. So I decided to install *Ghost* alongside *Wordpress*. This way, I can maintain my old posts until I, or someone else, hack out support for slugs with slashes in *Ghost*.

Now, all my existing posts can still be reached through their original permalinks. These posts are currently being served by *Wordpress*. However, all new blog posts, starting with this one, will be served by *Ghost*. You'll notice that both platforms still serve through the `/blog` subdirectory. This presented a more challenging engineering problem. I needed to find a way to serve `/blog/...` requests with their respective platforms (old posts with *Wordpress* and new posts with *Ghost*). I got kinda lucky and exploited a unique slug format in my *Wordpress* posts to achieve this goal. To find out more about how I ran both, *Ghost* and *Wordpress*, through the same subdirectory, check out this [post](http://thecalvinchan.com/blog/using-nginx-as-a-reverse-proxy/). 