## Introduction

In this tutorial, we would be creating a **EARN** (i.e ExpressJS, AngularJS, Redis and NodeJS) stack. We will use express-generator to setup basic structure of the nodejs app; on the frontend - we would be using AngularJS and create a single page application and, as a datastore - we will use Redis.

## Basic Setup
Before we get started with creating the stack, let's first setup the enviroment:
1. Install [NodeJS](https://nodejs.org/download "NodeJS").
2. Install Redis:
   - For Windows, download relevant [exe](https://github.com/rgl/redis/downloads "Redis on Windows").
   - For Mac, follow this [tutorial](http://jasdeep.ca/2012/05/installing-redis-on-mac-os-x/ "Redis on Mac")
   - For Linux, follow this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-redis "Redis on Linex")

## Let's Go
### Setup ExpressJS
####Create Skeleton
Open terminal in your working directory and execute the following commands:
```
npm install -g express-generator

express app_name

cd app_name && npm install
```
Above steps would create:
- public folder: for all the CSS and JS files
- routes folder: contains handler for route endpoint
- views folder
- app.js file: handles all the requests and routes
- package.json file: contains app information and dependencies.

To check that everything is working, execute `node ./bin/www`, and in browser open http://localhost:3000. 


Bydefault, the view engine is set as JADE but since we want to use AngularJS in front-end, we will use EJS instead:
 - In package.json, replace `"jade": "~1.9.2"` with `"ejs": "*"`
 - In app.js, replace `app.set('view engine', 'jade');` with
 > app.engine('html', require('ejs').renderFile);
 > app.set('view engine', 'html');
 - Execute npm install
 - Remove all the .jade files from view folder and create a basic index.html file.

#### Create Routes
Quoting from expressjs offical site:
> Routing refers to the definition of end points (URIs) to an application and how it responds to client requests.
> A `route` is a combination of a URI, a HTTP request method (GET, POST, and so on), and one or more handlers for the endpoint `app.METHOD(path, [callback...], callback)`.

Inorder to make our app more maintainable, we would have two main routers - one for rendering view which would serve AngularJS and other for handling our api calls.

For this, create `view.js` and `api.js` in routes folder and include them in app.js file and instruct app to use these two as routers
```javascript
var viewRoute = require('./routes/view')
	,apiRoute = require('./routes/api');

app.use('/',viewRoute);
app.use('/api',apiRoute);
```
Add the following in view.js file:
```javascript
var express = require('express');
module.exports = (function() {
	'use strict';
	var viewsRoute = express.Router();
	
	viewsRoute.get('/', function(req, res) {
		res.render('index');
	});

	return viewsRoute;
})();
```
Similarly, we can render any other view depending on the requesting url. Further, add the following in api.js:
```javascript
var express = require('express');

module.exports = (function() {
	'use strict';
	var api = express.Router();
	api.get('/',function(req,res){
		res.json({'key':'value'});
	});
	return api;
})();
```
Although right-now we are sending hard-coded values, we will be building on this to serve different request and send corredponding responses.

Again, To check that everything is working, execute `node ./bin/www`, and in browser open http://localhost:3000. This time index.html file would be rendered

### Setup AngularJS
We want to build a single page application so we will have a templates file `index.html` and inside that we would display different views based on path requested.
1. Create a module.js file in public folder (empty as of now).
2. In index.html file:
  - include [angular.js](//ajax.googleapis.com/ajax/libs/angularjs/1.3.14/angular.min.js) file (I prefer using google cdn) and [angular-route.js](https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0rc1/angular-route.min.js) file
  - include module.js file
  - Anyother css file you would use for styling (i.e. Bootstrap v3 or Angular Material)
  - Add `<ng-view></ng-view>` in the `<body>` tag and remove everything else.
  - Add `ng-app="earnfrontend" ng-controller="AppCtrl"` as attributes of <html> tag. ng-app directive defines an Angular app and ng-controller directive defines which application controller has access.

Uptil now we have created a template which we will apply to every view which is rendered. Template gives uniformity to overall application and makes it easy to manage all the required js/css files.

Now, create a html file in views folder and create a route in `view.js` which renders that html file:
```javascript
viewsRoute.get('/home', function(req, res) {
		res.render('home');
	});
```

Next, open the module.js file and add the following:
```javascript
var mainmodule = angular.module('earnfrontend', ['ngRoute']);
mainmodule.controller('AppCtrl', function($scope) { 
			});
mainmodule.config(
				function($routeProvider) {
					$routeProvider
						.when('/', {
							templateUrl: '/home',
							controller: 'AppCtrl'
						})
						.otherwise({
							redirectTo: '/'
						});
				});
```
Essentially, this initializes angular module, defines a controller and creates a [$routeProvider](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider) which loads a view in `<ng-view>` based on the path provided in templateUrl. In our case, `/home` goes to viewsRoute through app.js and renders `home.html` file.

Again run the app and you will see content from home.html.

### Setup Redis
In the working directory, to add the Redis client for nodejs, execute:
```
npm install --save redis
```
In the api.js file, include the redis module, and initialize it:
```javascript
var redis = require("redis"),
        client = redis.createClient(); // defaults to 127.0.0.1:6379
```
Add a middleware which just stores a new key:value pair in redis database:
```javascript
api.use(function(req,res,next){
		client.set("string key", "string val", redis.print);
		next();
	});
```
Now fire up the redis server, start redis client to Monitor the server and in browser open http://localhost:3000/api/. You will see that the command we wrote in middleware is executed and, 'string key' & 'string value' are stored in redis as a key:value pair.

DONE! We have created a basic setup of our **EARN** stack.

### Mix 'em Up
Now that we have everything setup, let's make them do the talking:
Create a textbox in `home.html` and a button
```
<div>
	<h2>This is home</h2>
	<div ng-controller="ListController">
		<form name="myListForm" >
		  New List Item : <input type="text" name="input" ng-model="listitem" required ng-trim="false">
		  <span class="error" ng-hide="myListForm.input.$error.required">
		    <button ng-click="sendToServer()">Send</button></span>
		 </form>
		 <li ng-repeat="item in items">{{item}}</li>
	</div>
</div>
```
In module.js, create a new ListController which sends the text from textbox to server on click of button and also fetches values from the list from server:

```javascript
mainmodule.controller('ListController', function($scope, $http) {
      $scope.sendToServer = function(){
      	$http.post('/api',{'item':$scope.listitem})
      		 .success(function(res){
      		 	getFromServer();
      		 });
      }
      getFromServer();
      function getFromServer(){
      	$http.get('/api')
      		 .success(function(res){
      		 	$scope.items = res;
      		 	console.log(res);
      		 });
      }
    });
```

On the server, handle GET and POST calls on path `"/"`. On received the call, the function will either extract items from list or push a new item on the list:
```javascript
api.get('/',function(req,res){
	client.lrange('mylist','0','-1',function(err,reply){
		res.json(reply);
	});
});

api.post('/',function(req,res){
	client.lpush('mylist',req.body.item);
	res.json({"result":"received"});
});
```

###End Note
This was just a basic setup tutorial and it does not even began to portray it's Power. I have created this tutorial to help fellow developers get over the initial barier of setting up the development stack and get started with creating amazing apps.
Since in most general cases, you need a persistent storage backing your application, you can use Redis as a persistence storage and there is a great article about it on their [website](http://redis.io/topics/persistence "Redis Persistence Storage").

##Happy Coding!!





.