---
title: 'Node.js Passport and Terms of Use: sign in to MyPortfolio'
date: 2017-11-08 17:49:20
tags:
---

In my application named [MyPortfolio](http://myportfolio.danilocarrabino.net) (built with Node.js/Express and MongoDB), I decided to use [Passport](http://www.passportjs.org/) as authentication middleware.

It is extremely flexible and modular, and it comes with a comprehensive set of strategies which support authentication using a _username and password_, _Google-OAuth_, _Facebook_, _Twitter_, and more.

I started developing a Passport authentication, by following a very interesting tutorial of [Scotch.io](https://scotch.io/tutorials/easy-node-authentication-setup-and-local), where it is clearly illustrated how to use _Passport_ to perform local authentication, _Facebook_, _Twitter_, _Google_ authentications, and even how to link all social accounts under one user account.

But something was missing in that tutorial, and many other _Passport_ tutorials I found on the web: how could I use _Passport_ middleware to perform a sign-up only after user viewed and accepted my application's Terms of Use?

Let me show you the architecture, the models and the operations flow I used (check also the code here _[MyPortfolio code](https://github.com/evildead/my-portfolio)_).

## Package.json

Here are the dependencies I'm using in package.json:
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

Here is the user model schema, which resembles the one in the _Scotch.io_ tutorial.
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
