# Chapter 10
In this chapter we will explore the continuous delivery development process with Node.js applications.
In all previous chapters we did an overview of how to develop various types of application, using different features of Node.js and the Express and Loopback frameworks, including using different databases as MongoDB and MySql.

On chapter 09, we saw how to make a deployment of our application using the command line and uploading the project directly to the cloud.
In this chapter we will see how to integrate some more tools into our development environment to dealing with unit tests and automated deployment, how to setup configuration variables to protect our database credentials and how to deploy using containers concept with Docker.

In this chapter we will cover:
* How dealing with CI solutions.
* How to configure MongoDB cloud instance and environment variables.
* How to integrate Github, Heroku and Codeship in build process.
* How to containers and Docker images.
* How to deploy an application using Docker.

# What we are building
In this chapter we will build an application with Express framework using some techniques already used in previous chapters as User session and user authentication with email and password using the passport middleware. We will also use MongoDB, Mongoose and Swig templates.
The result will be the following image:
- insert image BO5288_10_10.png

# Continuous Integration solutions
The workflow of Continuous Integration consist in 4 steps generally.

* Commit the code to a repository.
* The CI interface build the application.
* An then execute tests.
* Is all tests have success, the code goes to deployment.

The following figure illustrate this process.
- insert image BO5288_10_11.png

# Creating the baseline application
Let's start the application itself. First off we will create an application folder and add some root files.

## Adding the root files.
1- Create a folder called chapter-10.
2- Create a new file called package.json and add the following code:

    {
      "name": "chapter-10",
      "version": "1.0.0",
      "main": "server.js",
      "description": "Create an app for the cloud with Docker",
      "scripts": {
        "build": "npm-run-all build-*",
        "build-css": "node-sass public/css/main.scss > public/css/main.css",
        "postinstall": "npm run build",
        "start": "node server.js",
        "test": "mocha",
        "watch": "npm-run-all --parallel watch:*",
        "watch:css": "nodemon -e scss -w public/css -x npm run build:css"
      },
      "dependencies": {
        "async": "^1.5.2",
        "bcrypt-nodejs": "^0.0.3",
        "body-parser": "^1.15.1",
        "compression": "^1.6.2",
        "dotenv": "^2.0.0",
        "express": "^4.13.4",
        "express-flash": "0.0.2",
        "express-handlebars": "^3.0.0",
        "express-session": "^1.2.1",
        "express-validator": "^2.20.4",
        "method-override": "^2.3.5",
        "mongoose": "^4.4.8",
        "morgan": "^1.7.0",
        "node-sass": "^3.6.0",
        "nodemon": "^1.9.1",
        "npm-run-all": "^1.8.0",
        "passport": "^0.3.2",
        "passport-local": "^1.0.0",
        "swig": "^1.4.2"
      },
      "devDependencies": {
        "chai": "^3.5.0",
        "mocha": "^2.4.5",
        "sinon": "^1.17.3",
        "sinon-chai": "^2.8.0",
        "supertest": "^1.2.0"
      },
      "engines": {
        "node": "6.1.0"
      }
    }

3- Create a file called .env and add the following code:

    SESSION_SECRET='<SESSION_SECRET>'

    #MONGODB='<>'

    MONGODB='<MONGODB>'

Don't worry about the previous code, later in the chapter we will replace this code using environment variables.

> For security reasons never upload your credentials to OpenSource repositories, if you are working on a commercial project, even if you have a private repository. Is always recommended to use environment variable in production.

4- Create a file called Profile and add the following code:

    web: node server.js

As we already saw this file is responsible to make our application works on Heroku.
Also as we are using GIT source control is a good practice to include a .gitignore.

5- Create a file called .gitignore and add the following code:

    lib-cov
    *.seed
    *.log
    *.csv
    *.dat
    *.out
    *.pid
    *.gz
    *.swp

    pids
    logs
    results
    tmp
    coverage

    # API keys
    .env

    # Dependency directory
    node_modules
    bower_components
    npm-debug.log

    # Editors
    .idea
    *.iml

    # OS metadata
    .DS_Store
    Thumbs.db

## Creating config folder and files
1- At the root project folder, create a new folder called config.
2- Create a file called passport.js and add the following code:

    // load passport module
    var passport = require('passport');
    var LocalStrategy = require('passport-local').Strategy;
    // load up the user model
    var User = require('../models/User');

    passport.serializeUser(function(user, done) {
      // serialize the user for the session
      done(null, user.id);
    });

    passport.deserializeUser(function(id, done) {
      // deserialize the user
      User.findById(id, function(err, user) {
        done(err, user);
      });
    });

    // using local strategy
    passport.use(new LocalStrategy({ usernameField: 'email' }, function(email, password, done) {

      User.findOne({ email: email }, function(err, user) {
        if (!user) {
          // check errors and bring the messages
          return done(null, false, { msg: 'The email: ' + email + ' is already taken. '});
        }
        user.comparePassword(password, function(err, isMatch) {
          if (!isMatch) {
            // check errors and bring the messages
            return done(null, false, { msg: 'Invalid email or password' });
          }
          return done(null, user);
        });
      });

    }));

The previous code will take care of user authentication using the flask middleware for error messages as we saw on chapter 01.

## Creating controllers folder and files
As we are building a simple application we will have only two controllers, one for Users and another simple to home page.

1- At the root project folder, create a new folder called controllers.
2- Create a file called home.js and add the following code:

    // Render Home Page
    exports.index = function(req, res) {
      res.render('home', {
        title: 'Home'
      });
    };

3- Create a file called user.js.

Now let's add all the functions related to users as sign in, sign up, authorization, account and log out. We will adding each function following the previous one.

### Adding modules and authentication middleware.

1- Add the following code at controllers/user.js:

    // import modules
    var async = require('async');
    var crypto = require('crypto');
    var passport = require('passport');
    var User = require('../models/User');

    // authorization middleware
    exports.ensureAuthenticated = function(req, res, next) {
      if (req.isAuthenticated()) {
        next();
      } else {
        res.redirect('/login');
      }
    };

    // logout
    exports.logout = function(req, res) {
      req.logout();
      res.redirect('/');
    };

### Adding login GET and POST methods

1- Add the following code at controllers/user.js, right after the previous one:

    // login GET
    exports.loginGet = function(req, res) {
      if (req.user) {
        return res.redirect('/');
      }
      res.render('login', {
        title: 'Log in'
      });
    };

    // login POST
    exports.loginPost = function(req, res, next) {
      // validate login form fields
      req.assert('email', 'Email is not valid').isEmail();
      req.assert('email', 'Empty email not allowed').notEmpty();
      req.assert('password', 'Empty password not allowed').notEmpty();
      req.sanitize('email').normalizeEmail({ remove_dots: false });

      var errors = req.validationErrors();

      if (errors) {
        // Show errors messages for form validation
        req.flash('error', errors);
        return res.redirect('/login');
      }

      passport.authenticate('local', function(err, user, info) {
        if (!user) {
          req.flash('error', info);
          return res.redirect('/login')
        }
        req.logIn(user, function(err) {
          res.redirect('/');
        });
      })(req, res, next);
    };

### Adding signup GET and POST methods
1- Add the following code at controllers/user.js, right after the previous one:

    // signup GET
    exports.signupGet = function(req, res) {
      if (req.user) {
        return res.redirect('/');
      }
      res.render('signup', {
        title: 'Sign up'
      });
    };

    // signup POST
    exports.signupPost = function(req, res, next) {
      // validate sign up form fields
      req.assert('name', 'Empty name not allowed').notEmpty();
      req.assert('email', 'Email is not valid').isEmail();
      req.assert('email', 'Empty email is not allowed').notEmpty();
      req.assert('password', 'Password must be at least 4 characters long').len(4);
      req.sanitize('email').normalizeEmail({ remove_dots: false });

      var errors = req.validationErrors();

      if (errors) {
        // Show errors messages for form validation
        req.flash('error', errors);
        return res.redirect('/signup');
      }

      // Verify user email
      User.findOne({ email: req.body.email }, function(err, user) {
        if (user) {
          // if used, show message and redirect
          req.flash('error', { msg: 'The email is already taken.' });
          return res.redirect('/signup');
        }
        // create an instance of user model with form data
        user = new User({
          name: req.body.name,
          email: req.body.email,
          password: req.body.password
        });
        // save user
        user.save(function(err) {
          req.logIn(user, function(err) {
            res.redirect('/');
          });
        });
      });
    };

### Adding account GET and UPDATE methods
1- Add the following code at controllers/user.js, right after the previous one:

    // profile account page
    exports.accountGet = function(req, res) {
      res.render('profile', {
        title: 'My Account'
      });
    };

    // update profile and change password
    exports.accountPut = function(req, res, next) {
      // validate sign up form fields
      if ('password' in req.body) {
        req.assert('password', 'Password must be at least 4 characters long').len(4);
        req.assert('confirm', 'Passwords must match').equals(req.body.password);
      } else {
        req.assert('email', 'Email is not valid').isEmail();
        req.assert('email', 'Empty email is not allowed').notEmpty();
        req.sanitize('email').normalizeEmail({ remove_dots: false });
      }

      var errors = req.validationErrors();

      if (errors) {
        // Show errors messages for form validation
        req.flash('error', errors);
        return res.redirect('/pages');
      }

      User.findById(req.user.id, function(err, user) {
        // if form field password change
        if ('password' in req.body) {
          user.password = req.body.password;
        } else {
          user.email = req.body.email;
          user.name = req.body.name;
        }
        // save user data
        user.save(function(err) {
          // if password field change
          if ('password' in req.body) {
            req.flash('success', { msg: 'Password changed.' });
          } else if (err && err.code === 11000) {
            req.flash('error', { msg: 'The email is already taken.' });
          } else {
            req.flash('success', { msg: 'Profile updated.' });
          }
          res.redirect('/account');
        });
      });
    };

### Adding account DELETE method
1- Add the following code at controllers/user.js, right after the previous one:

    // profile DELETE
    exports.accountDelete = function(req, res, next) {
      User.remove({ _id: req.user.id }, function(err) {
        req.logout();
        req.flash('info', { msg: 'Account deleted.' });
        res.redirect('/');
      });
    };

Now we finished the controllers creation.

## Creating models folder and files.
1- At the root project folder, create a folder called models.
2- Create a new file called User.js and add the following code:

    // import modules
    var crypto = require('crypto');
    var bcrypt = require('bcrypt-nodejs');
    var mongoose = require('mongoose');

    // using virtual attributes
    var schemaOptions = {
      timestamps: true,
      toJSON: {
        virtuals: true
      }
    };

    // create User schema
    var userSchema = new mongoose.Schema({
      name: String,
      email: { type: String, unique: true},
      password: String,
      picture: String
    }, schemaOptions);

    // encrypt password
    userSchema.pre('save', function(next) {
      var user = this;
      if (!user.isModified('password')) { return next(); }
      bcrypt.genSalt(10, function(err, salt) {
        bcrypt.hash(user.password, salt, null, function(err, hash) {
          user.password = hash;
          next();
        });
      });
    });
    // Checking equais password
    userSchema.methods.comparePassword = function(password, cb) {
      bcrypt.compare(password, this.password, function(err, isMatch) {
        cb(err, isMatch);
      });
    };

    // using virtual attributes
    userSchema.virtual('gravatar').get(function() {
      if (!this.get('email')) {
        return 'https://gravatar.com/avatar/?s=200&d=retro';
      }
      var md5 = crypto.createHash('md5').update(this.get('email')).digest('hex');
      return 'https://gravatar.com/avatar/' + md5 + '?s=200&d=retro';
    });

    var User = mongoose.model('User', userSchema);

    module.exports = User;

## Creating public folder and files
In this example we are using the Bootstrap framework, the SASS version as we did on previous chapter. But this time we will store the source files in a different location, inside de public/css folder. Let's create the folder and files.

1- Inside the root project, create a folder called public.
2- Inside the public folder create three new folders called css/vendor/bootstrap.
3- Go to https://github.com/twbs/bootstrap-sass/tree/master/assets/stylesheets/bootstrap copy all content and paste into public/css/vendor/bootstrap folder.
4- Inside public/css/vendor folder, create a new file called _bootstrap.scss and add the following code:

    /*!
     * Bootstrap v3.3.6 (http://getbootstrap.com)
     * Copyright 2011-2015 Twitter, Inc.
     * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
     */

    // Core variables and mixins
    @import "bootstrap/variables";
    @import "bootstrap/mixins";

    // Reset and dependencies
    @import "bootstrap/normalize";
    @import "bootstrap/print";
    @import "bootstrap/glyphicons";

    // Core CSS
    @import "bootstrap/scaffolding";
    @import "bootstrap/type";
    @import "bootstrap/code";
    @import "bootstrap/grid";
    @import "bootstrap/tables";
    @import "bootstrap/forms";
    @import "bootstrap/buttons";

    // Components
    @import "bootstrap/component-animations";
    @import "bootstrap/dropdowns";
    @import "bootstrap/button-groups";
    @import "bootstrap/input-groups";
    @import "bootstrap/navs";
    @import "bootstrap/navbar";
    @import "bootstrap/breadcrumbs";
    @import "bootstrap/pagination";
    @import "bootstrap/pager";
    @import "bootstrap/labels";
    @import "bootstrap/badges";
    @import "bootstrap/jumbotron";
    @import "bootstrap/thumbnails";
    @import "bootstrap/alerts";
    @import "bootstrap/progress-bars";
    @import "bootstrap/media";
    @import "bootstrap/list-group";
    @import "bootstrap/panels";
    @import "bootstrap/responsive-embed";
    @import "bootstrap/wells";
    @import "bootstrap/close";

    // Components w/ JavaScript
    @import "bootstrap/modals";
    @import "bootstrap/tooltip";
    @import "bootstrap/popovers";
    @import "bootstrap/carousel";

    // Utility classes
    @import "bootstrap/utilities";
    @import "bootstrap/responsive-utilities";


### Creating a custom stylesheet
6- Inside public/css/ create a new file called main.scss and add the following code:

    // import bootstrap
    @import "vendor/bootstrap";

    // Structure
    html {
      position: relative;
      min-height: 100%;
    }

    body {
      margin-bottom: 44px;
    }

    footer {
      position: absolute;
      width: 100%;
      height: 44px;
      padding: 10px 30px;
      bottom: 0;
      background-color: #fff;
      border-top: 1px solid #e0e0e0;
    }

    .login-container {
      max-width: 555px;
    }

    // Warning
    .alert {
      border-width: 0 0 0 3px;
    }

    // Panels
    .panel {
      border: solid 1px rgba(160, 160, 160, 0.3);
      box-shadow: 0 1px 4px 0 rgba(0, 0, 0, 0.1);
    }

    .panel-heading + .panel-body {
      padding-top: 0;
    }

    .panel-body {
      h1, h2, h3, h4, h5, h6 {
        margin-top: 0;
      }
    }

    .panel-title {
      font-size: 18px;
      color: #424242;
    }

    // Form
    textarea {
      resize: none;
    }

    .form-control {
      height: auto;
      padding: 8px 12px;
      border: 2px solid #ebebeb;
      border-radius: 0;
      box-shadow: inset 0 1px 2px rgba(150, 160, 175, 0.1), inset 0 1px 15px rgba(150, 160, 175, 0.05);
    }

    .form-group > label {
      text-transform: uppercase;
      font-size: 13px;
    }

Don't worry about the node-sass building process now, we already setup a NPM task on package.json file at the beginning of this chapter.

### Creating the fonts folder and adding font files
1- Inside public folder create a new folder called fonts.
2- Go to https://github.com/twbs/bootstrap-sass/tree/master/assets/fonts/bootstrap , copy all the content and paste at public/fonts folder.

### Creating the JavaScript folder and files
1- Inside public folder, create a new folder called js.
2- Inside js folder, create a new folder called lib.
3- Inside js/lib create a new file called bootstrap.js.
4- Go to https://github.com/twbs/bootstrap-sass/blob/master/assets/javascripts/bootstrap.js, copy all the content and paste at public/js/lib/bootstrap.js file.
5- Create a new file called jquery.js.
6- Go to https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.js, copy all content and paste at public/js/lib/jquery.js file.

## Creating the views folder and files
Now we will create a very similar folder structure from chapter 01, the views folder will have the following directories:

    views/
        layouts/
        pages/
        partials/

### Adding layouts folder and file
1- Inside views/layouts create a new file called main.html and add the following code:

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Chapter-10</title>
    <title>{{title}}</title>
    <link rel="stylesheet" href="/css/main.css">
    </head>
    <body>
    {% include "../partials/header.html" %}
        {% block content %}
        {% endblock %}
    {% include "../partials/footer.html" %}
    <script src="/js/lib/jquery.js"></script>
    <script src="/js/lib/bootstrap.js"></script>
    <script src="/js/main.js"></script>
    </body>
    </html>

### Adding pages folder and files
1- Inside views/pages create a file called home.html and add the following code:

    {% extends '../layouts/main.html' %}

    {% block content %}
        <div class="container">
          {% if messages.success %}
            <div role="alert" class="alert alert-success">
              {% for item in messages.success %}
                <div>{{ item.msg }}</div>
              {% endfor %}
            </div>
          {% endif %}
          {% if messages.error %}
            <div role="alert" class="alert alert-danger">
              {% for item in messages.error %}
                <div>{{ item.msg }}</div>
              {% endfor %}
            </div>
          {% endif %}
          {% if messages.info %}
            <div role="alert" class="alert alert-info">
              {% for item in messages.info %}
                <div>{{ item.msg }}</div>
              {% endfor %}
            </div>
          {% endif %}
          <div class="app">
            <div class="jumbotron">
              <h1 class="text-center">Node 6 Bluenprints Book</h1>
                <p>This example illustrate how to build, test and deploy a Node.js application using: Github, Heroku, MOngolab, Codeship and Docker.</p>
              </div>
          </div>
        </div>
    {% endblock %}


2- Inside views/pages create a file called login.html and add the following code:

    {% extends '../layouts/main.html' %}

    {% block content %}
        <div class="login-container container">
          <div class="panel">
            <div class="panel-body">
              {% if messages.error %}
                <div role="alert" class="alert alert-danger">
                  {% for item in messages.error %}
                    <div>{{ item.msg }}</div>
                  {% endfor %}
                </div>
              {% endif %}
              <form method="POST">
                <legend>Welcome to login</legend>
                <div class="form-group">
                  <label for="email">Email</label>
                  <input type="email" name="email" id="email" placeholder="Email" class="form-control" autofocus>
                </div>
                <div class="form-group">
                  <label for="password">Password</label>
                  <input type="password" name="password" id="password" placeholder="Password" class="form-control">
                </div>
                <button type="submit" class="btn btn-primary btn-block">Sign in</button>
              </form>
            </div>
          </div>
          <p class="text-center">Don't have an account? <a href="/signup"><strong>Sign up</strong></a>, it's free.</p>
        </div>
    {% endblock %}

3- Inside views/pages create a file called profile.html and add the following code:

    {% extends '../layouts/main.html' %}

    {% block content %}
        <div class="container">
          <div class="panel">
            <div class="panel-body">
              {% if messages.success %}
                <div role="alert" class="alert alert-success">
                  {% for item in messages.success %}
                    <div>{{ item.msg }}</div>
                  {% endfor %}
                </div>
              {% endif %}
              {% if messages.error %}
                <div role="alert" class="alert alert-danger">
                  {% for item in messages.error %}
                    <div>{{ item.msg }}</div>
                  {% endfor %}
                </div>
              {% endif %}
              <form method="POST" action="/account?_method=PUT">
                <legend>Account Details</legend>
                <div class="form-group">
                    <label for="email">Email</label>
                    <input type="email" name="email" id="email" class="form-control" value="{{user.email}}">
                </div>
                <div class="form-group">
                    <label for="name">Name</label>
                    <input type="text" name="name" id="name" class="form-control" value="{{user.name}}">
                </div>
                  <br>
                <div class="form-group">
                    <button type="submit" class="btn btn-primary">Update Profile</button>
                </div>
              </form>
            </div>
          </div>
          <div class="panel">
            <div class="panel-body">
              <form method="POST" action="/account?_method=PUT">
                <legend>Change Password</legend>
                <div class="form-group">
                  <label for="password">New Password</label>
                    <input type="password" name="password" id="password" class="form-control">
                </div>
                <div class="form-group">
                  <label for="confirm">Confirm Password</label>
                    <input type="password" name="confirm" id="confirm" class="form-control">
                </div>
                <div class="form-group">
                    <button type="submit" class="btn btn-success">Change Password</button>
                </div>
              </form>
            </div>
          </div>
          <div class="panel">
            <div class="panel-body">
              <form method="POST" action="/account?_method=DELETE">
                <legend>Delete My Account</legend>
                <div class="form-group">
                  <p class="text-muted">It is irreversible action.</p>
                    <button type="submit" class="btn btn-danger">Delete</button>
                  </div>
              </form>
            </div>
          </div>
        </div>
    {% endblock %}

4- Inside views/pages create a file called signup.html and add the following code:

    {% extends '../layouts/main.html' %}

    {% block content %}
    <div class="login-container container">
      <div class="panel">
        <div class="panel-body">
          {% if messages.error %}
            <div role="alert" class="alert alert-danger">
              {% for item in messages.error %}
                <div>{{ item.msg }}</div>
              {% endfor %}
            </div>
          {% endif %}
          <form method="POST">
            <legend>Create an account</legend>
            <div class="form-group">
              <label for="name">Name</label>
              <input type="text" name="name" id="name" placeholder="Name" class="form-control" autofocus>
            </div>
            <div class="form-group">
              <label for="email">Email</label>
              <input type="email" name="email" id="email" placeholder="Email" class="form-control">
            </div>
            <div class="form-group">
              <label for="password">Password</label>
              <input type="password" name="password" id="password" placeholder="Password" class="form-control">
            </div>
            <button type="submit" class="btn btn-primary btn-block">Sign up</button>
          </form>
        </div>
      </div>
      <p class="text-center"> Already have an account? <a href="/login"><strong>Sign in</strong></a></p>
    </div>
    {% endblock %}

### Adding partials folder and files
1- Inside views/partials create a new file called footer.html and add the following code:

    <footer>
        <div class="container">
            <p>Node.js 6 Blueprints Book © 2016. All Rights Reserved.</p>
        </div>
    </footer>

2- Inside views/partials create a new file called header.html and add the following code:

    <nav class="navbar navbar-default navbar-static-top">
      <div class="container">
        <div class="navbar-header">
          <button type="button" data-toggle="collapse" data-target="#navbar" class="navbar-toggle collapsed">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a href="/" class="navbar-brand">N6B</a>
        </div>
        <div id="navbar" class="navbar-collapse collapse">
          <ul class="nav navbar-nav">
            <li class="{% if  title == 'Home' %}active{% endif %}"><a href="/">Home</a></li>
          </ul>
        <ul class="nav navbar-nav navbar-right">
        {% if user %}
          <li class="dropdown">
            <a href="#" data-toggle="dropdown" class="navbar-avatar dropdown-toggle">
              {% if user.picture %}
                <img src="{{user.picture}}">
              {% else %}
                <img src="{{user.gravatar}}">
              {% endif %}
              {% if user.name %}
                {{user.name}}
              {% else %}
                {{user.email}}
              {% endif %}
              <i class="caret"></i>
            </a>
            <ul class="dropdown-menu">
              <li><a href="/account">My Account</a></li>
              <li class="divider"></li>
              <li><a href="/logout">Logout</a></li>
            </ul>
          </li>
        {% else %}
          <li class="{% if  title == 'Log in' %}active{% endif %}"><a href="/login">Log in</a></li>
          <li class="{% if  title == 'Sign up' %}active{% endif %}"><a href="/signup">Sign up</a></li>
        {% endif %}
        </ul>
        </div>
      </div>
    </nav>



---
# Creating a Github or Bitbucket free account
You can choose what service to use, Github or Bitbucket doing the same thing, hosting public and private repositories of code.
The functionality of both are similar and both use the GIT as a source control. We will show how to use Github.

## Github

1- Go to https://github.com/join, fill the form and press the create button.
2- Choose Unlimited public repositories for free checkbox and press continue button.
3- On third step you must answer three questions or choose to skip this step, press submit button.

From here to forward you can read the guide or start a project.

> Note that you need to verify your email before start a project.

    git remote add origin https://github.com/fernandomonteirodev/n6b.git
    git push -u origin master

4- Press start a project button and fill the repository name as the following figure:
- Insert image BO5288_10_00.png

Later in the chapter we will see how to initialize a repository.

# Creating a Heroku free account
On previous chapter we deployed the application directly to Heroku using the Heroku toolbelt commands. So if don't have the Heroku toolbelt
go to https://toolbelt.heroku.com/ and get the version for your platform.

# Creating a Mongolab free sandbox account

1- Proceed to signup page, after that you must receive two emails from Mongolab one for welcome messages and another to verify your account, if you don't have one yet.
After you verify your account you will see the following image:
- Insert image BO5288_10_01.png

2- Click on Create new button.
3- Choose single-node tab.
4- At standard line panel, choose the first checkbox for sandbox.
5- Scroll down to end of the page and insert a database name: nb6 and click on: create new mongodb deployment button.

At the end of these five steps we can see the following screen:
- Insert image BO5288_10_02.png

## Creating an user and password for database
It' time to create an user and password to protect our database

1- Click on database name.
You will see the following warning, suggesting you to create an user and password.
- Insert image BO5288_10_03.png

2- Click on the Click Here link inside the warning message.
3- insert the following infos:

database username: nb6
database password: node6

4- Click on create user button.

## Getting the string connection

That's all, now we have a MongoDB instance running on MongoLab cloud service. And here's the connection string that we will use later in the chapter:

    mongodb://<user>:<password>@ds023074.mlab.com:23074/nb6

> No that you must replace the previous code with your own user and password.

# Initializing a git repository and push to Github

1- Open your terminal/shell inside root application folder and type the following command:

    git init

2- Add a remote to project, type on terminal/shell the following command::

    git remote add origin https://github.com/<your github account name>/n6b.git

> No that you must replace the previous code with your own github username.

3- Add all project files to source control, type on terminal/shell the following command:

    git add .

4- Commit the project changes, type on terminal/shell the following command::

    git commit -m "initial commit"

The last command is to upload all files to the Github repository that we created before.

5- Type on terminal/shell the following command:

    git push -u origin master

# Creating a Heroku application using Heroku Dashboard

1- On Heroku dashboard click on new button, and then click on create new app link.
2- Place the following name into app input name field: chapter-10
3- Click create app button.

## Linking Heroku application to your git repository
4- On Heroku dashboard click on chapter-10 project name.
5- Click on settings tab, scroll down the page to Domains and copy the Heroku domain url.

    chapter-10.herokuapp.com

Later we will use the app name to configure Codeship deployment pipeline.

## Adding environment variables to project
Now we need to create some environment variable to keep our database string safe.

1- On Heroku dashboard click on chapter-10 project name.
2- Click on settings tab.
3- On settings tab, click o reveal Config Vars button.
4- Add your own vars, as the following figure. On the left side place the variable name and the right side place the value.
- insert image BO5288_10_12.png

> Note that you must repeat this process on Codeship configuration project.

# Creating a Codeship free account
Codeship is a cloud service for continuous integration (CI) tool. It's pretty simple to create an account.

1- Go to https://codeship.com/sessions/new and use the sign in buttons at the right side, you can use your Github or Bitbucket account, just press your prefered button

2- Click on authorize application button.
You must see the following screen:
- Insert image BO5288_10_04.png

The next step is to click where you hosting your code. In this case we will click on Github icon. So we will see the following screen:
- Insert image BO5288_10_05.png

3- Copy and paste the github repository url(https://github.com/fernandomonteirodev/n6b.git) created on Github setup process and paste on: Repository Clone URL input

4- Press connect button

Now we have setup our development environment with three tools: Github + Codeship + Heroku. The next step is to create setup and test commands and add a pipeline.

## Adding environment variables to project
Now let's do the same we did on Heroku dashboard and add the same variables to Codeship.

1- Go to: https://codeship.com/projects/, select chapter-10 project.
2- Click on the project settings link at the top right conner, as the following figure:
- insert image BO5288_10_06.png

3- Click on Environment Variables link.
4- Add the Session and MongoDB variables and values, as we did on Heroku environment variables configuration and click Save configuration button.

## Creating setup and test commands on codeship project configuration

1- Paste the following code into setup commands textarea:

    # By default we use the Node.js version set in your package.json or the latest
    # version from the 0.10 release
    #
    # You can use nvm to install any Node.js (or io.js) version you require.
    # nvm install 4.0
    # nvm install 0.10
    npm install
    npm run build

2- Paste the following code into test commands textarea:

    npm test

3- Click on save and go to dashboard button.

## Creating the pipeline to deployment on Heroku
Well, we almost there, now we need to create a pipeline to integrate the build with our deployment environment on Heroku.

1- Click on project settings link on the right top conner, and then click on deployment link as the following figure:
- Insert image BO5288_10_06.png

2- On Enter branchname input, place the following name: master

3- Click on save pipeline setting button.

4- On add a deployment tab choose heroku banner.

Now we will fill the following input fields as the following image:
- Insert image BO5288_10_07.png

### Adding the API Heroku key into Codeship
To fill the previous image we need to execute some steps to get the necessary information.

5- Open a new browser window and go to Heroku dashboard at: https://id.heroku.com/login.

6- Click on your picture at the top right conner, and then click on account settings.

7- Scroll down the page on API key, click show API Key, as the following figure:
- Insert image BO5288_10_08.png

8- Insert your password and copy the API key.

9- Go back to Codeship browser window and paste the key on Heroku API key input field.
10- Name it your application as n6b.
11- Add the application url: http://chapter-10.herokuapp.com/
11- Click on Save deployment button.

This step completes our continuous integration. Every time we modify our code and send the changes to Github, the Codeship will run the code and the tests we set when we created the Node.js application early in this chapter.
At the end of testing, if everything ok our code is sent to the Heroku and will be available at: http://chapter-10.herokuapp.com/.

### following the tests and deploy steps on Codeship dashboard

1- Go to https://codeship.com/sessions/new
2- Login your account.
3- On top left side conner, click on select project link.
4- Click on n6b project name and you will see all your commit with a success, running or failed flag.

When we click on one of then, we can see the following image with a step by step process, as the following figure:
- Insert image BO5288_10_09.png

Here we have a successfully build as we can see the green check icon on right side of each step.


# Installing Docker and setup the application

Before we go further we need to understand what is Docker and the concept of containers, thinking in a very simple way, Docker create micro machines inside a isolated box to run your application don't matter what is your platform. Let's see what the official Docker site say's:

> Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server.

So let's install docker on your machine.

1- Go to https://docs.docker.com/ and follow the instructions for your platform.

> You can read more about container at this link: https://www.docker.com/what-docker#/VM

## Checking Docker version
Now is time to check the Docker version.

1- Open your terminal/shell and type the following commands:

    docker --version

    docker-compose --version

    docker-machine --version

## Creating a Dockerfile
To using our application in a container we need to create two files, a Dockerfile and a docker-compose.yml file to link our application container with a database.

1- Inside the root folder, create a new folder called Dockerfile and add the following code:

    FROM node:argon

    # Create app directory
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app

    # Install app dependencies
    COPY package.json /usr/src/app/
    RUN npm install

    ENV PORT 3000
    ENV DB_PORT_27017_TCP_ADDR db

    # Bundle app source
    COPY . /usr/src/app

    EXPOSE 3000
    CMD [ "npm", "start" ]


2- Inside the root folder, create a new file called docker-compose.yml and add the following code:

    app:
      build: .
      ports:
        - "3000:3000"
      links:
        - db

    db:
      image: mongo
      ports:
        - "27017:27017"


Before we go further, let's check some useful Docker commands:

* To list all containers:

    docker ps -a

* To list all images:

    docker images

* To remove a specific container:

    docker rm containername

* To remove all containers:

    docker rm $(docker ps -a -q)

* To remove a specific image:

   docker rmi imagename

* To remove all images:

   docker rmi $(docker images -q)

* To start a container:

    docker run containername

* To stop a container:

    docker stop containername

* To stop all containers

    docker stop $(docker ps -a -q)

We have more commands, but along with the chapter we will see other's.

## Creating a Docker image

1- Creating the Docker image for your project:

    docker build -t <your docker user name>/<projectname> .

At the end of the output on terminal we can see a message similar as: Successfully built c3bbc61f92a6 .Now let's check the image already created.


2- Checking the images, open your terminal/shell and type the following command:

    docker images


## Preparing and running the Docker image
Now let's test our Docker image. Before we procceed we need to make a small change in our application.

1- Open server.js file at the root folder and replace the following code:

    mongoose.connect('mongodb://' + (process.env.DB_PORT_27017_TCP_ADDR || process.env.MONGODB) + '/<database name>');

2- Now open .env file and replace the code for the following lines:


    SESSION_SECRET='ae37a4318f1218302e16e1516e4144df8a273798b151ca06062c142bbfcc23bc'

    MONGODB='localhost:27017'

> Note that step 1 and 2 using local credentials, different from what we do on deployment. So after configure the environment variables on Heroku and Codeship, remove the .env file from github tracking.

3- Open your terminal/shell and type the following command:

    docker pull mongo

The previous command will get a new image for MongoDB.

4- Start a container named db with the following command:

    docker run -d --name db mongo

5- Now we need to link one container to another, type the following command:

    docker run -p 3000:3000 --link db:db <your docker user name>/<projectname>

6- Go to localhost:3000 and you must see your application running.


## Uploading the project image to your Docker hub account

Now is time to upload your image to Docker hub, and make available to other users.

> You can read more about Docker hub at this link: https://docs.docker.com/docker-hub/


1- Go to https://cloud.docker.com/ create a free account.

2- After confirm your e-mail address go to https://cloud.docker.com sign in menu. You will see the following dashboard as the previous image, when click on repositories button and you will see it is empty.

- insert image BO5288_10_13.png


3- Open your terminal/shell and type the following command:

    docker login

Enter your credentials.

4- To upload the project to Docker hub, run the following command:

    docker push <your docker user name>/<projectname>

5- Go back to: https://cloud.docker.com/_/repository/list and you must see your repository published on Docker hub.

Docker is a powerful tool and must be more explored, but for this chapter we have enough knowledge to build Node.js applications running with MongoDB.


# Summary
At the end of this chapter you are capable to build and deploy an application using all the most modern technologies and tools that we have available at the moment.
We explored all resource to build an application with continuous delivery and integration combining Git source control, Github, Codeship, Heroku and Docker. We saw how to protect your project using environment variable for test and deploy, also we saw how to



