As we mentioned before on previous chapters, we can use the facilities from NPM (Node Package Manager) to replace Gulp and Grunt task manager, the most popular tools to dealing with fron-end dependencies. We can combine both tools, but in this chapter we will explore only the NPM and some commands that will help us create our application.

We will create npm tasks to lint, concatenate and minify Javascript files, optimize images, compile SASS stylesheets and deploy the application to a server in the cloud, just using the command line. Furthermore for this example we will use the loopback.io framework to create the application example with MongoDB as database.

In this chapter we will cover:
* How to create an application using only the Loopback.io CLI.
* How to install eslint, imagemin and browserify.
* How to create tasks to lint errors, concatenate JS files and optimize images.
* How to dealing with SASS import and compile.
* How to deploy an application to Heroku using the Heroku toolbelt.

# What we are building

For this chapter we will build a simple gallery application, very similar to chapter 04, but this time we will using a Restful API with loopback.io framework. We'll see how create building task using the NPM command line, and the final result will be very similar to following figure:
- Insert Image BO5288_09_01.png

# Creating the baseline application with slc command line

Although we already used the loopback framework, we strongly recommend that you install it again, to ensure that you will have the most updated version on your machine.

    npm install -g loopback

In this example we will not make many changes in the generated code, since our focus is to the create building tasks, but we will use some interesting features of the loopback framework using the command line.


1- Open your terminal/shell, and type the following command:

    slc loopback

2- Name it the application as chapter-09.

3- Choose: empty-server (An empty LoopBack API, without any configured models or datasources) and press enter.

Now we have created the application scaffold. Don't worry about the next commands suggested by the terminal output, we will discuss and see this commands later in the book.

## Adding a Datasource to project

Before we create our models as we did in Chapter 6, this time we will add the Datasource first. This is because we are using the command line to create the entire project. This means that we don't edit any file manually.
When we create the models before creating the Datasource, we need to add them to the Datasource later by manually editing the file: server/model-config.json

1- In your terminal/shell go to chapter-09 folder, and type the following command:

    slc loopback:datasource

2- Fill the following questions as the following figure:
- Insert Image BO5288_09_02.png

By default we don't need to setup user and password if we are working with MongoDB on localhost, don't worry about this now, later we will see how to change this configuration, to deploy our application.

## Creating application models

Now let's create the application models, for this example we are using two models,

1- Open your terminal/shell inside chapter-09 folder, and type the following command:

    slc loopback:model

Use the model name: Gallery

2- Fill the following questions as the following figure:
- Insert Image BO5288_09_03.png

After the second property, press enter.

3- Open your terminal/shell inside chapter-09 folder, and type the following command:

    slc loopback:model

Use the model name: Bike

4- Fill the following questions as the following figure:
- Insert Image BO5288_09_04.png

After the third property, press enter.

Don't worry about the relationship between the models at this time, we will see on next step.

## Adding relationship between application models

Let's define the relationship between our models, we will use two types of relations:

* hasmany (A Gallery can have many bikes)
* belongsTo (A Bike can have one gallery)

Remember we just trying to make some useful, but not complex, to illustrate the building process with NPM.

1- Open your terminal/shell inside chapter-09 folder, and type the following command:

    slc loopback:relation

2- Choose bike model and fill the questions as the following information:
- Insert Image BO5288_09_05.png    

3- Choose gallery model and fill the questions with the following information:
- Insert Image BO5288_09_06.png

So let's check if everything is wrote properly.

4- Open common/models/gallery.json and you will see the following highlighted code:

    {
      "name": "gallery",
      "plural": "galleries",
      "base": "PersistedModel",
      "idInjection": true,
      "options": {
        "validateUpsert": true
      },
      "properties": {
          ...
      },
      "validations": [],
      "relations": {
        "bikes": {
          "type": "hasMany",
          "model": "bike",
          "foreignKey": ""
        }
      },
      "acls": [],
      "methods": {}
    }

5- Open common/models/bike.json and you will see the following highlighted code:

    {
      "name": "bike",
      "base": "PersistedModel",
      "idInjection": true,
      "options": {
        "validateUpsert": true
      },
      "properties": {
        ...
      },
      "validations": [],
      "relations": {
        "gallery": {
          "type": "belongsTo",
          "model": "gallery",
          "foreignKey": ""
        }
      },
      "acls": [],
      "methods": {}
    }

Using only three commands we managed to create the basis for our sample application.

## Setting a static site

As we did on chapter 06, let's setup the client's folder as a static site.

1- Rename server/boot/root.js file to server/boot/_root.js

2- Add the following highlighted lines to server.boot/middleware.json:

    "routes": {
      "loopback#rest": {
        "paths": [
          "${restApiRoot}"
        ]
      }
    },
    "files": {
        "loopback#static": {
          "params": "$!../client"
      }
    },
    "final": {
      "loopback#urlNotFound": {}
    },
    "final:after": {
      "loopback#errorHandler": {}
    }

3- Inside ./client folder create a new file called index.html and add the following content:

    <!DOCTYPE html>
    <html>
    <head><title>Bikes Gallery</title></head>
    <body>
        <h1>Hello!</h1>
    </body>
    </html>

4- Open your terminal/shell and type the following command:

    npm start

5- Open your favorite browser and go to http://localhost:3000/
You must see the hello! message.

Also we have our Restful API at: http://localhost:3000/api/bikes and http://localhost:3000/api/galleries.

Now we will see how to re structure some directories to prepare the application for deployment in the cloud and using the building tasks.

# Refactoring the application folder

Our refactoring process includes two steps. First let's create a directory for the application sources file such as JavaScript, SASS and images files.
In the second step we will create some directories within the client folder to receive our scripts.

## Creating a source files directory
Let's create the source folder for images, libs. scripts and scss files.

### Creating the images folder
In this folder we will store the images before processing an optimization technique using imagemin.

1- Inside the root project create a folder called src.

2- Within src folder, create a folder called images.

3- Within images folder, create a folder called gallery.

4- Download the sample-images files for chapter-09 from packtpub website or at the official book repository on github and paste at gallery folder.

### Creating the libs folder
In this folder we will store some jQuery plugins

1- Inside src folder, create a folder called libs.

### Creating scripts
As we are using jQuery and some plugins we will need to write some code to use the jQuery libs, we will do it using this folder.

1- Inside src folder, create a folder called scripts.

### Creating the SASS folder.
The SASS folder will store the scss files. We are using the Bootstrap framework, for this example we will setup up the bootstrap framework using the SASS separated version, don't worry with this now, later in the chapter we will show to how get these files.

The SCSS folder will have the following structure:

    scss/
        vendor/
            mixins/


## Installing Bower
As we have seen in previous chapters, we will use the Bower to manager front-end dependencies.

1- Open your terminal/shell and type de following command:

    npm install bower -g

2- Create a file called .bowerrc and save at the root folder.

3- Add the following content to .bowerrc file:

    {
      "directory": "src/components",
      "json": "bower.json"
    }

4- Open your terminal/shell and type de following command:

    bower init

Fill the following questions as the following image:
- Insert Image BO5288_09_07.png


### Installing application dependencies
In this example we are using just one jQuery plugin, plus Bootstrap framework, so let's first install the Bootstrap using the Bower CLI.

1- Open your terminal/shell and type the following command:

  bower install bootstrap#v4.0.0-alpha --save

Just open the src/components folder to check bootstrap and jQuery folder's. Now we will install the jQuery fancybox plugin.

2- Open your terminal/shell and type the following command:

  bower install fancybox --save

So the src/components will have the following folders at this moment:

    src/
        components/
            bootstrap/
            fancybox/
            jquery/


### Create the SCSS folder structure to compile Bootstrap framework

1- Open src/components/bootstrap and copy all the contents from SCSS folder.

2- Paste the content inside src/scss/vendor folder.

3- Create a file called main.scss inside src/ folder and add the following content:

    // Project Style

    // Import Botstrap
    @import "vendor/bootstrap";

    //
    body {
      padding-top: 5rem;
    }
    .starter-template {
      padding: 3rem 1.5rem;
      text-align: center;
      @include clearfix
    }

    bootstrap.css file in your projects.

Many developers do not use the boostrap framework this way, some just use bootstrap.css or bootstrap.min.css file in your projects, it is ok, but when we use the framework, the way explained in the above lines, we can use all the framework resources in our own stylesheet, so we can use all mixins and variables within our stylesheet.

For example, the highlighted code came from boostrap mixins:

    .starter-template {
      padding: 3rem 1.5rem;
      text-align: center;
      @include clearfix
    }

## Refactoring the client folder
The client folder will have a pretty basic structure for any web application with folders to store CSS, JavaScript and images files.
For this example we will use the last stable version from AngularJS to create the pages of our application.

1- Inside client folder create the following folders.

    css/
    images/gallery/
    js/
    js/libs/
    js/scripts/
    views/

### Adding the application views

1- Inside client/src folder create a new file called home.html and add the following code:

    <div class="col-md-6" ng-repeat="item in vm.listProducts">
      <div class="card" >
        <img class="card-img-top" ng-src="{{ item.image }}" alt="Card image cap" width="100%">
        <div class="card-block">
          <h4 class="card-title">{{ item.name }}</h4>
          <p class="card-text">{{ item.description }}</p>
          <a ui-sref="galleries({itemId:item.id})" class="btn btn-secondary">View Gallery</a>
        </div>
      </div>
    </div>
    </div>

2- Inside client/src folder create a new file called galleries.html and add the following code:

    <div class="row">
      <div class="col-md-4" ng-repeat="item in vm.listProducts">
        <div class="card" >
          <a href="{{ item.image }}" class="fancybox" rel="gallery" title="{{ item.name }}">
              <img class="card-img-top" ng-src="{{ item.image }}" alt="{{ item.name }}" width="100%"/>
          </a>
          <div class="card-block">
            <h4 class="card-title">{{ item.name }}</h4>
            <p class="card-text">{{ item.model }} - {{ item.category }}</p>
          </div>
        </div>
      </div>
    </div>

3- Open client/index.html file and add the following content:

    <!DOCTYPE html>
    <html ng-app="bikesGallery">
    <head><title>Bikes Gallery</title></head>
    <link rel="stylesheet" href="css/main.css">
    <link rel="stylesheet" href="components/fancybox/source/jquery.fancybox.css">
    <body>
        <nav class="navbar navbar-fixed-top navbar-dark bg-inverse">
            <div class="container">
                <a class="navbar-brand" href="#">Chapter 09</a>
                <ul class="nav navbar-nav">
                    <li class="nav-item active"><a class="nav-link" href="/">Home <span class="sr-only">(current)</span></a></li>
                </ul>
            </div>
        </nav>

        <div class="container">
            <div id="title">
                <div class="starter-template">
                    <h1>Image Gallery</h1>
                    <p class="lead">Select a Gallery.</p>
                </div>
            </div>
            <div class="" ui-view>

            </div>
        </div>
    </div>

    <!-- Scripts at bottom -->
    <script src='js/libs/jquery.min.js'></script>
    <script src="js/libs/angular.js"></script>
    <script src="js/libs/angular-resource.js"></script>
    <script src="js/libs/angular-ui-router.js"></script>
    <script src="js/app.js"></script>
    <script src="js/app.config.js"></script>
    <script src="js/app.routes.js"></script>
    <script src="js/services.js"></script>
    <script src="js/controllers.js"></script>
    <script src="js/libs/libs.js"></script>
    <script src="js/scripts/scripts.js"></script>

    </body>
    </html>

## Installing AngularJS files
Now is time to install AngularJS files and create the Angular application. In this example we will explore the AngularJS SDK from Loopback framework later this this section.

1- Open your terminal/shell and type the following command:

    bower install angularjs#1.5.0 --save

2- Open your terminal/shell and type the following command:

    bower install angular-resource#1.5.0 --save

3- Open your terminal/shell and type the following command:

    bower install angular-ui-router --save

You can read more about AngularJS at this link: https://docs.angularjs.org/api

## Creating the AngularJS application

1- Inside client/js folder create a new file called app.js and add the following code:

    (function(){
        'use strict';

        angular
        .module('bikesGallery', ['ui.router','lbServices']);

    })();

Don't worry about lbServices dependency at this moment, later in this chapter we will see how to create this file using the AngularJS SDK tool built in on Loopback framework.

2- Inside client/js folder create a new file called app.config.js and add the following code:

    (function(){
        'use strict';

        angular
        .module('bikesGallery')
        .config(configure)
        .run(runBlock);

        configure.$inject = ['$urlRouterProvider', '$httpProvider', '$locationProvider'];

        function configure($urlRouterProvider, $httpProvider, $locationProvider) {

            $locationProvider.hashPrefix('!');

            // This is required for Browser Sync to work poperly
            $httpProvider.defaults.withCredentials = true;
            $httpProvider.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

            $urlRouterProvider
            .otherwise('/');

        }

        runBlock.$inject = ['$rootScope', '$state', '$stateParams'];

        function runBlock($rootScope, $state, $stateParams ) {

            $rootScope.$state = $state;
            $rootScope.$stateParams = $stateParams;
        }

    })();

3- Inside client/js folder create a new file called app.routes.js and add the following code:

    (function(){
        'use strict';

        angular
        .module('bikesGallery')
        .config(routes);

        routes.$inject = ['$stateProvider'];

        function routes($stateProvider) {
            $stateProvider
            .state('home', {
                url:'/',
                templateUrl: 'views/home.html',
                controller: 'HomeController',
                controllerAs: 'vm'
            })
            .state('galleries', {
                url:'/galleries/{itemId}/bikes',
                templateUrl: 'views/galleries.html',
                controller: 'GalleryController',
                controllerAs: 'vm'
            })
            ;
        }

    })();

4- Inside client/js folder create a new file called controllers.js and add the following code:

    (function(){
        'use strict';

        angular
            .module('bikesGallery')
            .controller('GalleryController', GalleryController)
            .controller('HomeController', HomeController);

        HomeController.$inject = ['Gallery'];
        function HomeController(Gallery) {
            var vm = this;

            vm.listProducts = Gallery.find();

            //console.log(vm.listProducts);
        }

        GalleryController.$inject = ['Gallery', '$stateParams'];
        function GalleryController(Gallery, $stateParams) {
            var vm = this;

            var itemId = $stateParams.itemId;
            //console.log(itemId);

            vm.listProducts = Gallery.bikes({
              id: itemId
            });

            //console.log(vm.listProducts);
        }

    })();

## Using the AngularJS SDK to create services.

1- Open your terminal/shell and type the following command:

    lb-ng ./server/server.js ./client/js/services.js

The previous command will create a file called services.js inside client/js folder with all methods (Create, Read, Update, Delete) and many more available on the Restful API created by the loopback framework.

You can check your local API running the command npm start on your terminal/shell at the root project folder. The API will available at http://0.0.0.0:3000/explorer.

The lbServices have the following methods:

    "create": {
      url: urlBase + "/galleries",
      method: "POST"
    },

    "createMany": {
      isArray: true,
      url: urlBase + "/galleries",
      method: "POST"
    },

    "upsert": {
      url: urlBase + "/galleries",
      method: "PUT"
    },

    "exists": {
      url: urlBase + "/galleries/:id/exists",
      method: "GET"
    },

    "findById": {
      url: urlBase + "/galleries/:id",
      method: "GET"
    },

    "find": {
      isArray: true,
      url: urlBase + "/galleries",
      method: "GET"
    },

    "findOne": {
      url: urlBase + "/galleries/findOne",
      method: "GET"
    },

    "updateAll": {
      url: urlBase + "/galleries/update",
      method: "POST"
    },

    "deleteById": {
      url: urlBase + "/galleries/:id",
      method: "DELETE"
    },

    "count": {
      url: urlBase + "/galleries/count",
      method: "GET"
    },

    "prototype$updateAttributes": {
      url: urlBase + "/galleries/:id",
      method: "PUT"
    },

    "createChangeStream": {
      url: urlBase + "/galleries/change-stream",
      method: "POST"
    },

To use one of this methods we just inject the factory into our controller as the following highlighted code:

  GalleryController.$inject = ['Gallery', '$stateParams'];
  function GalleryController(Gallery, $stateParams) {
    ...
  }

And to use the method like the following example:

  Gallery.create();
  Gallery.find();
  Gallery.upsert({ id: itemId });
  Gallery.delete({ id: itemId });

A simple and very useful service to dealing with all endpoints created in our application for all models that we have.

The first part of the application is already almost completed, however we still need to add some content to make it more pleasant.

Let's create some content. As already mentioned earlier, you can download the entire sample code from the book on the website of packtpub or directly in the book repository.

# Adding content to application

You can enter content in two ways, the first using the endpoints created by the application or by using the migration file.

In the following lines we will show how to use the second option, it may be a brief and interesting procedure for a book.

1- Inside server/boot/ folder, create a file called create-sample-models.js and add the following content:

    module.exports = function(app) {
        // automigrate for models, everytime the app will running, db will be replaced with this data.

    	app.dataSources.galleryDS.automigrate('gallery', function(err) {
        if (err) throw err;
    	// Simple function to create content
        app.models.Gallery.create(
            [
    			{
                    "name":"Bikes",
    				        "image": "images/gallery/sample-moto-gallery.jpg",
    				        "link": "bikes.html",
                    "description":"Old and Classic Motorcycles",
                    "id":"5755d253b4aa192e41a6be0f"
                },{
                    "name":"Cars",
    				        "image": "images/gallery/sample-car-gallery.jpg",
    				        "link": "cars.html",
                    "description":"Old and Classic Cars",
                    "id":"5755d261b4aa192e41a6be10"
                }

    		], function(err, galleries) {
                if (err) throw err;
    			// Show a success msg on terminal
                console.log('Created Motorcycle Gallery Model: \n', galleries);
        });
      });

      app.dataSources.galleryDS.automigrate('bike', function(err) {
        if (err) throw err;
    	// Simple function to create content
        app.models.Bike.create(
            [
                {
                    "name":"Harley Davidson",
    				        "image": "images/gallery/sample-moto1.jpg",
                    "model":"Knuklehead",
                    "category":"Custom Classic Vintage",
                    "id":"5755d3afb4aa192e41a6be11",
                    "galleryId":"5755d253b4aa192e41a6be0f"
                },{
                    "name":"Harley Davidson",
    				        "image": "images/gallery/sample-moto2.jpg",
                    "model":"Rare Classic",
                    "category":"Custom Classic Vintage",
                    "id":"5755d3e8b4aa192e41a6be12",
                    "galleryId":"5755d253b4aa192e41a6be0f"
                },{
                    "name":"Old Unknown Custom Bike",
    				        "image": "images/gallery/sample-moto3.jpg",
                    "model":"Custom",
                    "category":"Chopper",
                    "id":"5755d431b4aa192e41a6be13",
                    "galleryId":"5755d253b4aa192e41a6be0f"
                },{
                    "name":"Shadow Macchit",
    				        "image": "images/gallery/sample-car1.jpg",
                    "model":"Classic",
                    "category":"Old Vintage",
                    "id":"5755d43eb4aa192e41a6be14",
                    "galleryId":"5755d261b4aa192e41a6be10"
                },{
                    "name":"Buicks",
    				        "image": "images/gallery/sample-car2.jpg",
                    "model":"Classic",
                    "category":"Classic",
                    "id":"5755d476b4aa192e41a6be15",
                    "galleryId":"5755d261b4aa192e41a6be10"
                },{
                    "name":"Ford",
    				        "image": "images/gallery/sample-car3.jpg",
                    "model":"Corsa",
                    "category":"Hatch",
                    "id":"5755d485b4aa192e41a6be16",
                    "galleryId":"5755d261b4aa192e41a6be10"
                }

    		], function(err, bikes) {
                if (err) throw err;
    			// Show a success msg on terminal
                console.log('Created Bike Model: \n', bikes);
        });
      });
    };

Don't forget to delete this file after the first deploy to Heroku.

# Creating the building tasks

Now is the time to create our task using only the npm (Node Package Manager).
Before we begin it is important to keep in mind that the npm has two special commands that are invoked directly, they are: start and test. So we will use the run command to run all the other task we create.

Our goal in this section is:

* Copy some files from source directory to client directory.
* Verify error on JavaScript files.
* Compile SASS files from src/scss and save it on client/css folder.
* Optimize images from src/images/gallery to client/images/gallery.
* Concat JavaScript files.

## installing the dependencies

To accomplish the above tasks, we need to install some CLI (Command line Interface) tools.

1- Open your terminal/shell and type the following commands:

  npm install copy-cli --save-dev

  npm install -g eslint

  npm install eslint --save-dev

  npm install -g node-sass

  npm install browserify --save-dev

  npm intall -g imagemin-cli

  npm install -g imagemin

Our purpose in this example is to show how to use the building tools, so we will not go too deep into each of them.

But before we go any further, let's setup the JavaScript validator eslint.

You can read more about eslint at this link: http://eslint.org/

2- Inside the root project, create a file called .eslintrc.json and add the following code:

  {
      "env": {
          "browser": true
      },
      "globals": {
          "angular": 1,
          "module": 1,
          "exports": 1
      },
      "extends": "eslint:recommended",
      "rules": {
          "linebreak-style": [
              "error",
              "unix"
          ],
          "no-mixed-spaces-and-tabs": 0,
          "quotes": 0,
          "semi": 0,
          "comma-dangle": 1,
          "no-console": 0
      }
  }

## Creating the copy task
We will create the task before inserting them into our package.json file, this way is easier to understand the procedure of each.

The copy tasks will be the following:

* Copy jQuery file.
* Copy AngularJS main library.
* Copy AngularJS resources library.
* Copy AngularJS ui-router library.

So we need to copy these files from source folder to client folder.

* "copy-jquery": "copy ./src/components/jquery/dist/jquery.js > ./client/js/libs/jquery.js",
* "copy-angular": "copy ./src/components/angular/angular.js > ./client/js/libs/angular.js",
* "copy-angular-resource": "copy ./src/components/angular-resource/angular-resource.js > ./client/js/libs/angular-resource.js",
* "copy-angular-ui-router": "copy ./src/components/angular-ui-router/release/angular-ui-router.js > ./client/js/libs/angular-ui-router.js",

The last copy task will execute all the other's copy tasks:

  "copy-angular-files": "npm run copy-angular && npm run copy-angular-resource && npm run copy-angular-ui-router",

Don't worry about to running the copy task at this moment, later in the chapter we will execute one by one.  

## Creating the SASS task.
The SASS task will be very simple, we will just compile the scss files and insert into client/css folder.

* "build-css": "node-sass --include-path scss src/scss/main.scss   client/css/main.css",

## Creating the linting task
We will use the .eslintrc.json configuration to apply to all JavaScript files at client/js folder.

* "lint-js": "eslint client/js/*.js --no-ignore",

## Creating the image optimization task
Another important task on any web application is to optimize all the image files, for performance reasons.

* "imagemin": "imagemin src/images/gallery/* --o client/images/gallery",

## Creating the concatenate task

* "concat-angular-js": "browserify ./src/libs/angular.js ./src/libs/angular-resource.js ./src/libs/angular-ui-router.js > client/js/libs/libs.js",
* "concat-js-plugins": "browserify src/libs/*.js -o client/js/libs/libs.js",
"concat-js-scripts": "browserify src/scripts/*.js -o client/js/scripts/scripts.js",

The last concat task is to execute all the other's concat tasks:

* "prepare-js": "npm run concat-js-plugins && npm run concat-js-scripts"

## Creating the build task
The build task is just the execution of each of the previous steps in a single task.

* "build": "npm run lint-js && npm run copy-angular-files && npm run build-css && npm run prepare-js"

Now let's add all tasks to package.json file.

1- Open package.json file and add the following highlighted code:

  {
    "name": "chapter-09",
    "version": "1.0.0",
    "main": "server/server.js",
    "scripts": {
      "start": "node .",
      "pretest": "eslint .",
      "posttest": "nsp check",
      "copy-jquery": "copy ./src/components/jquery/dist/jquery.js > ./client/js/libs/jquery.js",
      "copy-angular": "copy ./src/components/angular/angular.js > ./client/js/libs/angular.js",
      "copy-angular-resource": "copy ./src/components/angular-resource/angular-resource.js > ./client/js/libs/angular-resource.js",
      "copy-angular-ui-router": "copy ./src/components/angular-ui-router/release/angular-ui-router.js > ./client/js/libs/angular-ui-router.js",
      "copy-angular-files": "npm run copy-angular && npm run copy-angular-resource && npm run copy-angular-ui-router",
      "build-css": "node-sass --include-path scss src/scss/main.scss   client/css/main.css",
      "lint-js": "eslint client/js/*.js --no-ignore",
      "imagemin": "imagemin src/images/gallery/* --o client/images/gallery",
      "concat-angular-js": "browserify ./src/libs/angular.js ./src/libs/angular-resource.js ./src/libs/angular-ui-router.js > client/js/libs/libs.js",
      "concat-js-plugins": "browserify src/libs/*.js -o client/js/libs/libs.js",
      "concat-js-scripts": "browserify src/scripts/*.js -o client/js/scripts/scripts.js",
      "prepare-js": "npm run concat-js-plugins && npm run concat-js-scripts",
      "build": "npm run lint-js && npm run copy-angular-files && npm run build-css && npm run prepare-js"
    },
    "dependencies": {
      "compression": "^1.0.3",
      "cors": "^2.5.2",
      "helmet": "^1.3.0",
      "loopback": "^2.22.0",
      "loopback-boot": "^2.6.5",
      "loopback-component-explorer": "^2.4.0",
      "loopback-connector-mongodb": "^1.15.2",
      "loopback-datasource-juggler": "^2.39.0",
      "node-sass": "^3.7.0",
      "serve-favicon": "^2.0.1"
    },
    "devDependencies": {
      "browserify": "^13.0.1",
      "copy-cli": "^1.2.1",
      "eslint": "^2.12.0",
      "eslint-config-standard": "^5.3.1",
      "eslint-plugin-standard": "^1.3.2",
      "imagemin-cli": "^3.0.0",
      "nsp": "^2.1.0"
    },
    "repository": {
      "type": "",
      "url": ""
    },
    "license": "MIT",
    "description": "chapter-09",
    "engines": {
      "node": "5.0.x"
    }
  }

Tests on node.js applications should receive special attention and we could not show all about testing in just one chapter.

# Deploy to Heroku cloud
The first step to deploy our application is to create an account on Heroku.

1- Go to https://signup.heroku.com/?c=70130000001x9jFAAQ, and create a free account.

2- Download the Heroku toolbelt for your platform at: https://toolbelt.heroku.com/

Now you must have the Heroku toolbelt at your machine, to test it:

3- Open your terminal/shell and type the following command:

  heroku --help

The terminal output list all the possible things to do with Heroku CLI.

Now you must have git source control installed on your machine, if you don't have yet, the this instructons: https://git-scm.com/

## Creating a Heroku application
Now we will create an application and send it to your account newly created in Heroku.

1- Create a file called .Procfile and save it at the root project folder.

2- Paste the following code into .Procfile file:

  web: slc run

3- Open your terminal/shell and type the following command:

  git init

The previous command initialize a git repository.

  git add .

The git add command adding all files to version tracking.

  git commit -m "initial commit"

The git commit send all the files to version control.

### Creating a Procfile

  web: slc run

Now is time to login into you account and send all the project to cloud.

4- Open your terminal/shell and type the following command:

  heroku login

Insert you username and password.

5- Open your terminal/shell and type the following command:

  heroku apps:create --buildpack https://github.com/strongloop/strongloop-buildpacks.git

The previous command will use the strongloop-buildpacks to configure and deploy a Loopback application.

### Creating a deploy.sh file
Finally we will create our deploy file.

1- Create a folder called bin at the root folder.

2- Inside bin folder create a file called deploy.sh

3- Add the following code to bin/deploy.sh file:

  #!/bin/bash

  set -o errexit # Exit on error

  npm run build # Generate the bundled Javascript and CSS

  git push heroku master # Deploy to Heroku

4- Add the following highlighted code to package.json file.

{
  "name": "chapter-09",
  "version": "1.0.0",
  "main": "server/server.js",
  "scripts": {
    "start": "node .",
    "pretest": "eslint .",
    "posttest": "nsp check",
    "copy-jquery": "copy ./src/components/jquery/dist/jquery.js > ./client/js/libs/jquery.js",
    "copy-angular": "copy ./src/components/angular/angular.js > ./client/js/libs/angular.js",
    "copy-angular-resource": "copy ./src/components/angular-resource/angular-resource.js > ./client/js/libs/angular-resource.js",
    "copy-angular-ui-router": "copy ./src/components/angular-ui-router/release/angular-ui-router.js > ./client/js/libs/angular-ui-router.js",
    "copy-angular-files": "npm run copy-angular && npm run copy-angular-resource && npm run copy-angular-ui-router",
    "build-css": "node-sass --include-path scss src/scss/main.scss   client/css/main.css",
    "lint-js": "eslint client/js/*.js --no-ignore",
    "imagemin": "imagemin src/images/gallery/* --o client/images/gallery",
    "concat-angular-js": "browserify ./src/libs/angular.js ./src/libs/angular-resource.js ./src/libs/angular-ui-router.js > client/js/libs/libs.js",
    "concat-js-plugins": "browserify src/libs/*.js -o client/js/libs/libs.js",
    "concat-js-scripts": "browserify src/scripts/*.js -o client/js/scripts/scripts.js",
    "prepare-js": "npm run concat-js-plugins && npm run concat-js-scripts",
    "build": "npm run lint-js && npm run copy-angular-files && npm run build-css && npm run prepare-js",
    "deploy": "./bin/deploy.sh"
  },
  "dependencies": {
    ...
  },
  "devDependencies": {
    ...
  },
  "repository": {
    ...
  },
  "license": "MIT",
  "description": "chapter-09",
  "engines": {
    "node": "5.0.x"
  }
}


Now every time you make a commit with some changes and type the npm run deploy.
The engine will start the deploy .sh file and upload all the commited changes to Heroku cloud service.

5- Open your terminal/shell and type the following command:

  npm run deploy

If you facing errors about permission, do it the following:

5a- Open your terminal/shell inside the bin folder and type the following command:

  chmod 755 deploy.sh

By default Heroku cloud service will create a url for your application like this:
https://some-name-1234.herokuapp.com/

At the end of the output on terminal you will see some very similar to the following lines:

remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing...
remote:        Done: 79.7M
remote: -----> Launching...
remote:        Released v13
remote:        https://yourURL-some-23873.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/yourURL-some-23873.git

And the final result will be our sample application deploied to Heroku cloud service.

Just got to: https://yourURL-some-23873.herokuapp.com/ and you must see the following result:
- Insert Image BO5288_09_01.png

When you click on bikes view galley button, you will see the bike gallery as the following figure:
- Insert Image BO5288_09_10.png
