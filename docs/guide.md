
### Creating An Application

The _express.Server_ now inherits from _http.Server_, however
follows the same idiom by providing _express.createServer()_ as shown below. This means
that you can utilize Express server's transparently with other libraries.

    var app = require('express').createServer();
	
	app.get('/', function(req, res){
		res.send('hello world');
	});
	
	app.listen(3000);

### Configuration

Express supports arbitrary environments, such as _production_ and _development_. Developers
can use the _configure()_ method to setup needs required by the current environment. When
_configure()_ is called without an environment name it will be run in _every_ environment 
prior to the environment specific callback.

In the example below we only _dumpExceptions_, and respond with exception stack traces
in _development_ mode, however for both environments we utilize _methodOverride_ and _bodyDecoder_.

    app.configure(function(){
		app.use('/', connect.methodOverride());
		app.use('/', connect.bodyDecoder());
	});
	
	app.configure('development', function(){
		app.use('/', connect.errorHandler({ dumpExceptions: true, showStack: true }));
	});
	
	app.configure('production', function(){
		app.use('/', connect.errorHandler());
	});

For internal and arbitrary settings Express provides the _set(key[, val])_, _enable(key)_, _disable(key)_ methods:

    app.configure(function(){
		app.set('views', __dirname + '/views');
		app.set('views');
		// => "... views directory ..."
		
		app.enable('some feature');
		// same as app.set('some feature', true);
		
		app.disable('some feature');
		// same as app.set('some feature', false);
	});

### Settings

Express supports the following settings out of the box:

  * _home_ Application base path used with `res.redirect()`
  * _views_ Root views directory defaulting to **CWD/views**
  * _view engine_ Default view engine name for views rendered without extensions
  * _reload views_ Reloads altered views, by default watches for _mtime_ changes with
      with a 5 minute interval. Example: `app.set('reload views', { interval: 60000 });`

### Routing

Express utilizes the HTTP verbs to provide a meaningful, expressive routing API.
For example we may want to render a user's account for the path _/user/12_, this
can be done by defining the route below. The values associated to the named placeholders,
are passed as the _third_ argument, which here we name _params_.

    app.get('/user/:id', function(req, res, params){
		res.send('user ' + params.id);
	});

A route is simple a string which is compiled to a _RegExp_ internally. For example
when _/user/:id_ is compiled, a simplified version of the regexp may look similar to:

    \/user\/([^\/]+)\/?

Literal regular expressions may also be passed for complex uses:

	app.get(/^\/foo(bar)?$/, function(){});

Below are some route examples, and the associated paths that they
may consume:

     "/user/:id"
     /user/12

     "/users/:id?"
     /users/5
     /users

     "/files/*"
     /files/jquery.js
     /files/javascripts/jquery.js

     "/file/*.*"
	 /files/jquery.js
	 /files/javascripts/jquery.js
	
	 "/user/:id/:operation?"
	 /user/1
	 /user/1/edit
	
	 "/products.:format"
	 /products.json
	 /products.xml

	 "/products.:format?"
	 /products.json
	 /products.xml
	 /products

### Passing Route Control

We may pass control to the next _matching_ route, by calling the _fourth_ parameter,
the _next()_ function. When a match cannot be made, control is passed back to Connect.

	app.get('/users/:id?', function(req, res, params){
		if (params.id) {
			// do something
		} else {
			next();
		}
	});
	
	app.get('/users', function(req, res, params){
		// do something else
	});

### Middleware

The Express _Plugin_ is no more! middleware via [Connect](http://github.com/extjs/Connect) can be
passed to _express.createServer()_ as you would with a regular Connect server. For example:

	var connect = require('connect'),
		express = require('express');

    var app = express.createServer(
		connect.logger(),
		connect.bodyDecoder()
	);

### ServerRequest#send(body|status[, headers|status[, status]])

The `res.send()` method is a high level response utility allowing you to pass
objects to respond with json, strings for html, arbitrary _Buffer_s or numbers for status
code based responses. The following are all valid uses:

     res.send(new Buffer('wahoo'));
     res.send({ some: 'json' });
     res.send('<p>some html</p>');
     res.send('Sorry, cant find that', 404);
     res.send('text', { 'Content-Type': 'text/plain' }, 201);
     res.send(404);

By default the _Content-Type_ response header is set, however if explicitly
assigned through `res.send()` or previously with `res.header()` or `res.contentType()`
it will not be set again.

### ServerResponse#redirect(url[, status])

Redirect to the given _url_ with a default response _status_ of 302.

    res.redirect('/', 301);
    res.redirect('/account');
    res.redirect('http://google.com');
    res.redirect('home');
    res.redirect('back');

Express supports "redirect mapping", which by default provides _home_, and _back_.
The _back_ map checks the _Referrer_ and _Referer_ headers, while _home_ utilizes
the "home" setting and defaults to "/".

### Server#redirect(name, val)

For use with `res.redirect()` we can map redirects at the application level as shown below:

    app.redirect('google', 'http://google.com');

Now in a route we may call:

   res.redirect('google');

We may also map dynamic redirects:

    app.redirect('comments', function(req, res, params){
        return '/post/' + params.id + '/comments';
    });

So now we may do the following, and the redirect will dynamically adjust to
the context of the request. If we called this route with _GET /post/12_ our
redirect _Location_ would be _/post/12/comments_.

    app.get('/post/:id', function(req, res){
        res.redirect('comments');
    });