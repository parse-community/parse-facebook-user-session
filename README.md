parse-facebook-user-session
===========================

A Cloud Code module to facilitate logging into an express website with Facebook.

If you'd like to require users to be logged into Facebook to access pages
on your site, it's easy using the `parseFacebookUserSession`
middleware module. First set up the `parseExpressCookieSession`
as described in
<a href="https://parse.com/docs/hosting_guide#webapp-users">our express docs</a>.

    var parseExpressHttpsRedirect = require('parse-express-https-redirect');
    var parseExpressCookieSession = require('parse-express-cookie-session');
    var parseFacebookUserSession = require('cloud/parse-facebook-user-session');

    // ... Configure the express app ...

    app.use(parseExpressHttpsRedirect());  // Require user to be on HTTPS.
    app.use(express.bodyParser());
    app.use(express.cookieParser('YOUR_SIGNING_SECRET'));
    app.use(parseExpressCookieSession({ cookie: { maxAge: 3600000 } }));

Then use the `parseFacebookUserSession` middleware on any page
that you want to require Facebook login on. Whenever the page is visited, if
the user has not logged into your app with Facebook, they will be redirected
to Facebook's site to start an OAuth flow. Once they get a login token from
Facebook, they will be redirected back to an endpoint on your site. In the
example below, the endpoint is `/login`. The middleware module
takes care of handling that endpoint for you, authenticating the user and
redirecting them back to the original page they tried to visit. To enable this
on every page, use the `app.use` express method.</p>

    app.use(parseFacebookUserSession({
      clientId: 'YOUR_FB_CLIENT_ID',
      appSecret: 'YOUR_FB_APP_SECRET',
      redirectUri: '/login',
      scope: 'user_friends',
    }));

If you'd like to only require Facebook Login on certain pages, you can include
the middleware in your routing commands.

    var fbLogin = parseFacebookUserSession({
      clientId: 'YOUR_FB_CLIENT_ID',
      appSecret: 'YOUR_FB_APP_SECRET',
      redirectUri: '/login',
      scope: 'user_friends',
    });

    // To handle the login redirect.
    app.get('/login', fbLogin, function(req, res) {});

    app.get('/stuff', fbLogin, function(req, res) {
      // This page requires Facebook login.
    });

You can access the user on any page with `Parse.User.current`.

    app.get('/', function(req, res) {
      var user = Parse.User.current();

      res.render('hello', {
        message: 'Congrats, you are logged in, ' + user.get('username') + '!'
      });
    });

You should also provide an endpoint to let the user log out. This is as simple
as calling `Parse.User.logOut()`.

    app.get('/logout', function(req, res) {
      Parse.User.logOut();
      res.render('hello', { message: 'You are now logged out!' });
    });

As a side effect of using this module, you will see objects created in your
app's data for the `ParseFacebookTokenRequest` class. You can
safely ignore this class. It is used to keep track of the CSRF protection
tokens and redirect URLs of users who are currently in the process of
logging in.

In order for the `ParseFacebookTokenRequest` to be created, you may need to
enabled client class creation in your app's settings, if you've previously
disabled it. Once the `ParseFacebookTokenRequest` class is created, you should
go into the [class-level permissions](http://blog.parse.com/2014/07/07/parse-security-ii-class-hysteria/) for the class and disable all permissions. This cloud module uses
master key access to access the tables.

-----

As of April 5, 2017, Parse, LLC has transferred this code to the parse-community organization, and will no longer be contributing to or distributing this code.
