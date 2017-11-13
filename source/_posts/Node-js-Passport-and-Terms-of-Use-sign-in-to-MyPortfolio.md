---
title: 'Node.js Passport and Terms of Use: sign in to MyPortfolio'
date: 2017-11-08 17:49:20
tags:
---

In my application named [MyPortfolio](http://myportfolio.danilocarrabino.net) (built with Node.js/Express and MongoDB), I decided to use [Passport](http://www.passportjs.org/) as authentication middleware.

It is extremely flexible and modular, and it comes with a comprehensive set of strategies which support authentication using a _username and password_, _Google-OAuth_, _Facebook_, _Twitter_, and more.

I started developing a Passport authentication, by following a very interesting tutorial of [Scotch.io](https://scotch.io/tutorials/easy-node-authentication-setup-and-local), where it is clearly illustrated how to use _Passport_ to perform local authentication, _Facebook_, _Twitter_, _Google_ authentications, and even how to link all social accounts under one user account.

But there was something lacking in that tutorial, and many other _Passport_ tutorials I found on the web: how could I use _Passport_ middleware to perform a sign-up only after user viewed and accepted my application's Terms of Use?

Let me show you the architecture, the models and the operations flow I used (check also the code here _[MyPortfolio code](https://github.com/evildead/my-portfolio)_).

## Package.json

Here are the dependencies I'm using in _[package.json](https://github.com/evildead/my-portfolio/blob/master/package.json)_:
```js
// package.json

...
  "dependencies": {
    "async": "^2.5.0",
    "bcrypt-nodejs": "0.0.3",
    "body-parser": "^1.18.2",
    "connect-flash": "^0.1.1",
    "cookie-parser": "^1.4.3",
    "dotenv": "^4.0.0",
    "ejs": "^2.5.7",
    "express": "^4.14.0",
    "express-ejs-layouts": "^2.3.1",
    "express-session": "^1.15.6",
    "express-validator": "^4.2.1",
    "fs-extra": "^4.0.2",
    "mongoose": "^4.11.13",
    "multer": "^1.3.0",
    "passport": "^0.4.0",
    "passport-facebook": "^2.1.1",
    "passport-google-oauth": "^1.0.0",
    "passport-local": "^1.0.0",
    "passport-twitter": "^1.0.4",
    "summernote-nodejs": "^1.0.4"
  }
...
```

Because of I decided to let users log in only with their Google accounts in MyPortfolio, the dependencies of interest in this post are:
* _express_ is the framework.
* _express-session_ to handle the session in Express framework.
* _ejs_ is the templating engine.
* _mongoose_ is the object modeling for our MongoDB database.
* _passport_ and _passport-google-oauth_ for Google authentication.
* _connect-flash_ allows to pass session flashdata messages.
* _async_ lets easily perform asynchronous operations.
* _fs-extra_ empowers Node.js fs library with Promises.

## User model

Here is the _[user model schema](https://github.com/evildead/my-portfolio/blob/master/app/models/user.js)_, which resembles the one in the _Scotch.io_ tutorial.
```js
// user.js

...
var userSchema = mongoose.Schema({

    local            : {
        email        : String,
        password     : String,
    },
    facebook         : {
        id           : String,
        token        : String,
        email        : String,
        name         : String
    },
    twitter          : {
        id           : String,
        token        : String,
        displayName  : String,
        username     : String
    },
    google           : {
        id           : String,
        token        : String,
        email        : {type: String, index: true},
        name         : {type: String, index: true},
        imageUrl     : String,
        eslug        : String
    },
    mustacceptterms  : {
        type: Boolean,
        default: 'false'
    }
});
...
```

You can notice that in the userSchema I added another boolean parameter, named _mustacceptterms_: this one will be true until the user will have explicitly confirmed to have read and accepted the Terms of Use.

## Application setup

_[server.js](https://github.com/evildead/my-portfolio/blob/master/server.js)_ contains the setup of the different packages, included _Express_, _Passport_, _Session_ and _Mongoose_

```js
// server.js

// load environment variables
require('dotenv').config();

// grab our dependencies
const express = require('express'),
    app = express(),
    port = process.env.PORT || 8080
    expressLayouts = require('express-ejs-layouts'),
    mongoose = require('mongoose'),
    passport = require('passport'),
    bodyParser = require('body-parser'),
    session = require('express-session'),
    cookieParser = require('cookie-parser'),
    flash = require('connect-flash'),
    expressValidator = require('express-validator');

// configure our application
// set sessions and cookie parser
app.use(cookieParser());
app.use(session({
    secret: process.env.SECRET,
    cookie: {maxAge: 8*60*60*1000}, // 8 hours
    resave: false,      // forces the session to be saved back to the store
    saveUninitialized: false    // don't save unmodified sessions
}));
app.use(flash());

// tell express where to look for static assets
app.use(express.static(__dirname + '/public'));

// set ejs as templating engine
app.set('view engine', 'ejs');
app.use(expressLayouts);

// connect to the database
mongoose.connect(process.env.DB_URI);

// use bodyParser to grab info from a json
app.use(bodyParser.json({limit: '50mb'}));
// use bodyParser to grab info from a form
app.use(bodyParser.urlencoded({limit: '50mb', extended: true}));

// express validator -> validate form or URL parameters
app.use(expressValidator());

app.use(passport.initialize());
app.use(passport.session()); // persistent login sessions

require('./config/passport')(passport); // pass passport for configuration

// set the routes
app.use(require('./app/routes'));

// start our server
app.listen(port, () => {
    console.log(`App listening on http://localhost:${port}`);
});
```

## Passport Configuration

The _passport_ object is passed as parameter to _[config/passport.js](https://github.com/evildead/my-portfolio/blob/master/config/passport.js)_ where the Google OAuth strategy will be configured:

```js
// config/passport.js

// load only the Google strategy, because the login to My-Portfolio
// will be only possible by using the Google account
var GoogleStrategy = require('passport-google-oauth').OAuth2Strategy;

// load up the user model
var User = require('../app/models/user');

const initUserFolderStructure = require('../app/utilities').initUserFolderStructure;

module.exports = function(passport) {

    // =========================================================================
    // passport session setup ==================================================
    // =========================================================================
    // required for persistent login sessions
    // passport needs ability to serialize and unserialize users out of session

    // used to serialize the user for the session
    passport.serializeUser(function(user, done) {
        done(null, user.id);
    });

    // used to deserialize the user
    passport.deserializeUser(function(id, done) {
        User.findById(id, function(err, user) {
            done(err, user);
        });
    });

    // =========================================================================
    // GOOGLE ==================================================================
    // =========================================================================
    passport.use(new GoogleStrategy({

        clientID          : process.env.GOOGLEAUTH_clientID,
        clientSecret      : process.env.GOOGLEAUTH_clientSecret,
        callbackURL       : process.env.GOOGLEAUTH_callbackURL,
        passReqToCallback : true // allows us to pass in the req from our route (lets us check if a user is logged in or not)

    },
    ...
```

As shown above, _passport_ will use only the Google Oauth strategy, and the interesting part regards the callback invoked after authentication succeeds:

```js
...
function(req, token, refreshToken, profile, done) {
    // check if the user is already logged in
    if (!req.user) {

        User.findOne({ 'google.id' : profile.id }, function(err, user) {
            if (err) {
                return done(err);
            }

            if (user) {
                // if there is a user id already but no token (user was linked at one point and then removed)
                if (!user.google.token) {
                    user.google.token = token;
                    user.google.name  = profile.displayName;
                    user.google.email = profile.emails[0].value; // pull the first email
                    user.google.imageUrl = profile.photos[0].value; // pull the first image

                    initUserFolderStructure(profile.id, () => {
                        user.save(function(err) {
                            if (err) {
                                throw err;
                            }
                            return done(null, user);
                        });
                    });
                }

                if(!user.mustacceptterms) {
                    initUserFolderStructure(profile.id, () => {
                        user.save(function(err) {
                            if (err) {
                                throw err;
                            }
                            return done(null, user);
                        });
                    });
                }
                else {
                    return done(null, user);
                }
            }
            // brand new user
            else {
                var newUser = new User();

                newUser.google.id       = profile.id;
                newUser.google.token    = token;
                newUser.google.name     = profile.displayName;
                newUser.google.email    = profile.emails[0].value; // pull the first email
                newUser.google.imageUrl = profile.photos[0].value; // pull the first image
                newUser.mustacceptterms = true;     // the new user must accept terms of use before creating the account on MyPortfolio
                
                newUser.save(function(err) {
                    if (err) {
                        throw err;
                    }
                    return done(null, newUser);
                });
            }
        });
    }
    else {
        // user already exists and is logged in, we have to link accounts
        var user = req.user; // pull the user out of the session

        user.google.id       = profile.id;
        user.google.token    = token;
        user.google.name     = profile.displayName;
        user.google.email    = profile.emails[0].value; // pull the first email
        user.google.imageUrl = profile.photos[0].value; // pull the first image

        initUserFolderStructure(profile.id, () => {
            user.save(function(err) {
                if (err) {
                    throw err;
                }
                return done(null, user);
            });
        });
    }

}));
```

If a user logs in to MyPortfolio for the first time (_'A brand new user'_), he will be saved to the database with the flag _mustacceptterms_ set to _true_, and no folder structure will be created.

Even if the user had previously logged in without accepting the Terms of Use, he would just be considered authenticated, but the flag _mustacceptterms_ would remain set to _true_.

So I added two different functions to determine if user is really logged-in, or if he's just authenticated, and the flag _mustacceptterms_ is the determiner.

## Handling routing

The _[routes.js](https://github.com/evildead/my-portfolio/blob/master/app/routes.js)_ contains the authentication logic (and much more):

```js
// routes.js

// create a new express Router
const express = require('express'),
    router = express.Router(),
    multer = require('multer'),
    multerupload = multer({
        dest: 'uploads/',
        limits: { fieldSize: 25 * 1024 * 1024 }
    }),
    mainController = require('./controllers/main.controller'),
    authController = require('./controllers/auth.controller'),
    portfoliosController = require('./controllers/portfolios.controller');

const fs = require('fs');

// route middleware to make sure a user is logged in
function isLoggedIn(req, res, next) {
    // if user is authenticated in the session and the terms of use have been accepted, pass the control to the "next()" handler
    if (req.isAuthenticated() && !req.user.mustacceptterms) {
        return next();
    }

    // otherwise redirect him to the home page
    res.redirect('/');
}

// route middleware to make sure a user is temporarily logged in to accept terms of use
function isLoggedInToAcceptTerms(req, res, next) {
    //console.log(req.user);
    // if user is authenticated in the session and the terms of use have not been accepted, pass the control to the "next()" handler
    if (req.isAuthenticated() && req.user.mustacceptterms) {
        return next();
    }

    // otherwise redirect him to the home page
    res.redirect('/');
}

// export router
module.exports = router;

// define routes
// main routes
router.get('/', mainController.showHome);

// route for showing the "Terms of Use" page
router.get('/termsofuse', mainController.showTermsOfUse);

// Google OAuth authentication
router.get('/auth/google', passport.authenticate('google', { scope : ['profile', 'email'] }));

// the callback after Google has authenticated the user
router.get('/auth/google/callback',
    passport.authenticate('google', {failureRedirect : '/'}),    // on failure Google authentication
    function(req, res) {        // on correct Google authentication
        // User must accept the terms of use before creating the account
        if(req.user.mustacceptterms) {
            fs.readFile('./public/assets/termsOfUse.html', (err, data) => {
                if (err) throw err;
                res.render('pages/acceptTermsOfUse', {
                    user : req.user,
                    termscontent: data
                });
            });
        }
        // User has already accepted the terms of use
        else {
            res.redirect('/portfolios/editPortfolioInfos');
        }
});

// route for showing the profile page
router.get('/account', isLoggedIn, authController.showAccount);

// route for handling the terms of use acceptance
router.post('/accepttermsofuse', isLoggedInToAcceptTerms, authController.processAcceptTermsOfUse);
...
```

The utility function _isLoggedIn_ determines if a user is logged in the application.

The utility function _isLoggedInToAcceptTerms_ determines if a user is just authenticated (but still never accepted the Terms of Use).

The route '/auth/google' uses passport authentication method (which was previously configured with the Google Oauth2 Strategy) to authenticate the user with his Google account.

The magic happens for the route '/auth/google/callback', which represents the callback invoked after the Google authentication procedure. I used a 'failureRedirect' to the home page when the authentication fails, and a _Custom function_ on correct Google authentication.

Thanks to this custom function I can determine whether the Terms of Use haven't been previously accepted, and in that case redirect the user to the Terms of Use page: from there the user will have the chance to read and accept the Terms and complete the sign-in process.
