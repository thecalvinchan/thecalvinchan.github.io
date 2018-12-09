---
title: Auto-loading Backbone views with RequireJS
layout: post
permalink: auto-loading-backbone-views-with-requirejs
published: true
---
I'm nearing the critical period to cram for my Algorithms final tomorrow at 8 AM. But in honor of my impeccable study habits, here's a quick write up about (my approach with) auto-loading Backbone views with RequireJS.

If you've touched any bit of front-end web development, you should have no problem identifying with the endless chunk of `<script>` elements at the end of each `<body>`. And if you're using templates to render your html templates, you end up with one of two solutions:

1. You include all of your front-end controllers/views regardless of whether or not those particular views are used.
2. You spend a lot of time figuring out a less than trivial solution to identify which views to load for a particular page.

The first solution is less than ideal because you would essentially be loading a lot of useless code that wouldn't necessarily be used. So you're left with solution 2, which is to figure out how to explicitly inject certain scripts. One way to do this is through server-side logic. You can determine which scripts are needed on the server side and explicitly add `<script>` tags to include such scripts per template render. But that's a bit ugly (in my opinion) because then you start messing with front-end logic on the back-end.

To keep things isolated, I prefer to handle front-end dependency injection on the front-end. A super helpful library is [RequireJS](http://requirejs.org/). RequireJS allows you to define dependencies within each javascript file you create. Thus, if one of your javascript files have dependencies in another file, there's no need to worry about including the other file's `<script>` tag first.

![RequireJS](/blog/content/images/2014/Dec/Screenshot_2014_12_18_20_55_35.png)

However, RequireJS itself still doesn't solve the issue of loading an explicit view for a particular page (it only takes care of loading that view's dependencies). I found that an easy way to do this is to emulate Rails-like routing through RequireJS.

If you're familiar with Rails like architecture, you know that controllers and controller actions are invoked with a heavy dependency on the requested route. Not quite following what I mean?

`GET /home/index` would implicitly call the `indexAction` on the `homeController` of a traditional Rails application. I've grown to like this type of structure, and have always wanted to implement something similar on the front-end. So without further ado, here's a sample of code:

File Directory:

	/
    views/
      home/
        index.js
      login/
      	index.js
    app.js
    main.js
    vendor/
      jquery/
      underscore/
      backbone/
      requirejs/

main.js

	require.config({
    	// put your RequireJS configuration here
        // not important for our sample, but here for reference
        paths: {
            "jquery": "vendor/jquery/src/jquery",
            "underscore": "vendor/underscore/underscore-min",
            "backbone": "vendor/backbone/backbone",
        },
        shim: {
            "backbone": {
                deps: ["jquery", "underscore"],
                exports: "Backbone"
            },
        },
        waitSeconds: 10
    });
    
    require(['app'], function(App) {
    	// defines app.js as a dependency and
        // invokes the initialize method inside app.js
    	App.initialize();
    });
    
app.js

	var route = window.location.pathname;
    var queryParams = route.indexOf('?') === -1 ? null : route.indexOf('?');
    if (queryParams) {
        route = route.substring(1,queryParams);
    } else {
        route = route.substring(1);
    }
    route = route.split('/');
    if (route[0] == '') {
        route = [];
    }
    
    // If a controller is defined in the route, we load that controller, otherwise we load the home controller. If an action is defined, we load that action, otherwise we load the index action
    var controller = route.length > 0 ? route[0] : 'home';
        action = route.length > 1 ? route[1] : 'index';
    
    var routeView = ['views',controller,action].join('/');
    
    define([
    	'backbone',
        routeView
    ], function(Backbone, RouteView) {
        return {
            initialize: function() {
                new RouteView();
            }
        };
    });
    
The interesting part is the first few lines of `app.js`. Basically, we inspect the `window.location` of the user's browser to determine the route. We then parse the route into a controller (or in this case, a view) and an action.

So what's the benefit of injecting your scripts this way? Let's see how we would typically inject Backbone views without our custom RequireJS method.

	<!doctype html>
    <html>
    <head>...</head>
    <body>
    ...
        <script type="text/javascript" src="vendor/jquery/src/jquery.js"></script>
        <script type="text/javascript" src="vendor/underscore/underscore-min.js"></script>
        <script type="text/javascript" src="vendor/backbone/backbone.js"></script>
        <!-- Now depending on which page we are on, we have to determine which view to load, or load all of them (which is very poor practice)
        <script type="text/javascript" src="views/home/index.js">
        <script type="text/javascript" src="views/login/index.js">

    </body>
    </html>

As typical of any traditional web page, there is a chunk of `<script>` elements at the bottom of the page. Not only is this a mess to read, but if at any point, the order of which the scripts are included are mixed up, then you may run into an error.

With RequireJS and the custom solution that we cooked up, everything becomes much simpler.

	<!doctype html>
    <html>
	<head>
    ...
    <script data-main="main" src="vendor/requirejs/require.js"></script>
    </head>
    <body>
	</body>
	</html>
    
That's it! That's all you have to do. RequireJS will handle all of the dependency injection for you. Additionally, the custom view loader that we wrote will load an explicit view depending on which page you are on. This means that if you are on `/`, then the `/views/home/index.js` view will be loaded, and only that view will be loaded, nothing else. If you are on `/login`, then the `/views/login/index.js` view will be loaded. Full-blown front-end dependency injection and Rails-like view/controller loading, without any additional code in your html templates.

I've been meaning to write about how I approached this problem for a while, but never really had time to piece it all together. Although I think this is a good approach to this problem, it is not the only approach. There are a multitude of other ways to solve dependency injection issues, and which method is the best really comes down to the preference of the developer. If you have any ideas/comments, feel free to drop me a comment below.

Now back to NP-Complete...
    