---
title: Using Nginx as a reverse-proxy for Node and Apache
layout: post
permalink: using-nginx-as-a-reverse-proxy
published: true
---
I recently migrated my blog from *Wordpress* to *Ghost*, but I wanted to maintain my old blog posts on *Wordpress* (check [this](/blog/a-ghostly-upgrade) out for more background information). As such, I needed to configure my server to run the two different web platforms through a similar route-subdirectory. Here's how I did it:

##The environment

I installed *Wordpress* on a new Ubuntu EC2 instance, in the directory `/var/www/blog/`. After importing my of my old posts, my *Wordpress* posts followed this slug format:

	http://thecalvinchan.com/blog/%year/%month/%date/%title/
    
*Ghost* was also installed on the same EC2 instance. To maintain as much consistency as I could, I still wanted to serve *Ghost* posts through the `/blog` subdirectory. My *Ghost* posts follow this slug format:

	http://thecalvinchan.com/blog/%title/
    
Since all http requests go through port 80, I needed to find a way to route certain requests to *Apache* (which ran *Wordpress*) and certain requests to *Node* (which ran *Ghost*). I decided to use *Nginx* to proxy requests to their respective platforms.

(I decided to install another http server to proxy requests to their respective servers. However, you can easily achieve this same result by simply configuring *Apache* to proxy certain requests to *Node*)

##The configuration

####Apache

First, we must configure *Apache* to listen on requests through a different port (port 80 must be open for *Nginx* to listen to). 

File: `/etc/apache2/ports.conf`

	// - denotes delete line and + denotes add line
    - NameVirtualHost *:80
	- Listen 80
	+ NameVirtualHost *:8080
	+ Listen 8080

File: `/etc/apache2/sites-available/%your-site-configuration-file`

	- <VirtualHost *:80>
    + <VirtualHost *:8080>

What we've done is changed *Apache* to listen on port 8080 instead of 80. We should be done with all *Apache* configurations, assuming the prior configuration was default.

####Node

By default, *Ghost* configures *Node* to listen on port 2368. You don't need to change this.

####Nginx

Since we've changed *Apache* to listen on port 8080, Nginx can now listen to all http requests on port 80. We now need to configure it to proxy certain requests to *Apache* and other requests to *Node*. Go ahead and just copy and paste the following into your *Nginx* site configuration file. I've commented certain sections to explain what the code does.

File: `/etc/nginx/sites-available/%your-site-configuration-file`

	server {
      listen 80;
      root /var/www/;
      index index.php index.html index.htm;
      server_name example.com;
      
      // By default, all files are served through Nginx
      location / {
        try_files $uri $uri/ $uri/index.php /index.php;
      }
      
      // If the route uri matches a Wordpress uri or is a php file, proxy to Apache
      location ~ (/blog/[0-9]+/|/blog/wp-.*/|\.php$) {
        proxy_set_header	X-Real-IP	$remote_addr;
        proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
        proxy_set_header	Host		$http_host;
        proxy_set_header	X-NginX-Proxy	true;
        proxy_pass		http://127.0.0.1:8080;
        proxy_redirect off;
      }
      
      // If the route uri matches a Ghost uri, proxy to Node
      location ~ /blog/? {
        proxy_set_header 	X-Real-IP 	$remote_addr;
        proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
        proxy_set_header	Host		$http_host;
        proxy_set_header	X-NginX-Proxy	true;
        
        proxy_pass		http://127.0.0.1:2368;
        proxy_redirect off;
      }
      
      location ~ /\.ht {
        deny all;
      }
	}

The two lines of interest are:

	location ~ (/blog/[0-9]+/|/blog/wp-.*/|\.php$)
    
and 

	location ~ /blog/?
    
These two lines use Perl formatted regular expressions to match certain requests. Because my Wordpress slugs start with the year and follow this format:

	http://thecalvinchan.com/blog/%year/%month/%date/%title/
    
we can use a regular expression to match URIs with `/blog/[0-9]+/` and proxy those requests to *Apache*. Additionally, we proxy all *Wordpress* specific requests matching `blog/wp-.*/` to *Apache* as well. Assuming *Ghost* slugs don't start with a date, we can proxy all the other `/blog` requests to *Node*.

##Where this falls short

This solution is very dependent on the fact that my *Wordpress* slugs followed the `/blog/%year/%month/%date/%title` format. Because I am in control of my newer *Ghost* slugs, I can ensure that there will be no cross-match between the new slugs and the old slugs. However, this solution can be compromised if any new *Ghost* slug consists of only numeric characters. Alas, this is only a short-term solution to running both *Wordpress* and *Ghost*. Ideally, in the future, I can import my existing *Wordpress* posts to *Ghost* and simplify the http stack.

    



