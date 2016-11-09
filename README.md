**Node install and setup of project**

	Go to: https://nodejs.org/en/ and download and install node. 
	
	Open a terminal window, create and navigate to a new directory named for your project, run ```npm init```, and follow the prompts. 

**Setup git**

	If you haven't already, download and install git from here: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git.
	
	Set up a remote git repo in GitHub or any other place that you choose.
	
	Run these in the terminal from your project folder:
	
	```
	git init
	git add -A
	git remote add origin <insert remote repo URL>
	git commit -m "first commit"
	git push -u origin master
	```
		
**Installation of Node Packages**

	```
	Run these commands in your project folder:
	npm install jquery --save
	npm install express --save
	npm install jasmine -g --save-dev
	npm install jasmine-jquery --save-dev
	npm install karma-cli -g
	npm install karma --save-dev
	npm install karma-jasmine --save-dev
	npm install karma-jasmine-jquery --save-dev
	npm install karma-chrome-launcher --save-dev
	```
	
	These are packages that we will be using in our project.
	jQuery is for manipulating the DOM of our page.
	Express is a basic web server.
	Jasmine and Jasmine-jQuery are javascript unit testing tools.
	Karma is a tool for running jasmine tests in a browser environment.
	
	Next run:
	```karma init``` 
	karam init will walk you though a set of questions. Leave the defaults for all the options except...
	
	*When asked, "What is the location of your source and test files?" enter these options: *
	  ```
	  node_modules/jquery/dist/jquery.js
      src/*.js
      spec/**/*.js
    ```
    
    
    When you are done karma.conf.js will be created for you.
    Open it in an editor and make sure karma.conf.js contains the following frameworks and plugins:
    
    ```
    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['jasmine-jquery','jasmine'],
    
    // Which plugins to enable
    plugins: [
      'karma-jasmine-jquery',
      'karma-jasmine',
      'karma-chrome-launcher',
    ]```
	
	Run:
	```jasmine init``` which will create a default folder structure for you and a jasmine config file.
	
	Next create a new file ".gitignore" and add "node_modules/" to it.

**Start writing code**
	
	Add the following directories to your project folder, "src", and "public". 
	Create src/app.js, spec/appSpec.js, server.js and public/index.html
	
	Your directory should look like this:

	|-- .gitignore
	|-- karma.conf.js
	|-- package.json	
	|-- server.js
	|-- src
		|-- app.js
	|-- spec
		|--support
			|--jasmine.json
		|-- appSpec.js
	|-- public
		|-- index.html

	
	Add this markup to index.html:
	
	```<!DOCTYPE html>
    <html lang="en">

        <head>
            <meta charset="utf-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <meta name="description" content="">
            <meta name="author" content="">

            <title>CI Example</title>
            
        </head>
        <body>
			hi
        	<script src="scripts/jquery.js"></script>
        	<script src="src/app.js"></script>
        </body>
    </html>```
    
    Add this to server.js so that the files we are creating are available to be served up
    when we make http requests:
    
	```var express = require('express');
    var app = express();
    var path    = require("path");

	app.use('/scripts', express.static(__dirname + '/node_modules/jquery/dist'));
    app.use('/src', express.static(__dirname + '/src/'));
    app.use('/', express.static(__dirname + '/public/'))

    app.get('/', function (req, res) {
        res.sendFile(path.join(__dirname+'/index.html'));
    });

    app.listen(process.env.PORT || 5000, function() {
    	console.log('Example app listening on port 5000!');
    });```

    Add this to the scripts section of package.json:
    
    ```"start": "node server.js"``` 
    
	on the terminal, from your project directory, run "npm start". If you navigate to 
	localhost:5000 in a browser you should see our page load up
	
	
**Write a Test**

	Now that we have the basic project set up lets do something a little more interesting.
	Lets create a test that asserts we read some input fields from the screen when a method 
	is called.

	In appSpec.js add the following code:
	
	```
	'use strict'


	describe("Read Input", function(){
    	it("reads the value from our input field", function(){
        	$("body").append("<input id='name' value='blah'><input>");
	        var inputReader = new InputReader();
  			expect(inputReader.readInput()).toEqual("blah");
	        $("#name").remove();
    	});
	});
	```
	
	Run: 
	```karma start``` 
	Our test fail will fail.

**Write Code to Pass Test**
	
	In app.js add the following code: 
	
	```
	function InputReader(){
    	this.readInput = function(){
        	    return $("#name").val();
        	}
	}

	// Export node module.
	if ( typeof module !== 'undefined' && module.hasOwnProperty('exports') )
	{
    	module.exports = InputReader;
	}
	```
	
	back on the command line you should see karma detect the changes and automatically re-run
	the tests. They should now be green.

**Commit your Changes to Git**
	
	Run the following from the terminal:
	```
	git add .
	git commit -m 'initial project setup and basic test/functionality'
	git push
	```
	
**Setup CircleCI:**

	Create a CircleCI account by using your github account.
	Click add project and select your github account to browse repositories.
	Click "build project" on the repository we have been working in.
	You should see CircleCI immediately kick off a build for the project. It will fail because there are
	a few things we need to override that Circle is providing as a default.
	
	Create a new file named circle.yml at the root of your project and add the following:
	
	```
	machine:
		node:
    		version: 6.1.0

	dependencies:
		override:
    		- npm install --dev

	test:
		override: 
    		- npm run test-once
    ```
    
    We want to run the tests using a newer version of node than the default, we want to install
    all our dev dependencies before we try and run tests and we want to use a special test command
    so that karma doesn't hang the build while it's watching for file changes. 
    
    We also need to update the scripts in our package.json to this:
    
    ```
    "scripts": {
    	"test": "karma start",
	    "test-once": "karma start --single-run",
	    "start": "node server.js"
	}
	```

	commit and push again. You should see CircleCI automatically detect the push and run your tests.
	if all is well the tests should be green.
	

**Create Heroku App**

	Create a heroku account.
	Create a new app. 
	Add settings to CircleCI to make it so on succesful build our code will be pushed to heroku:
	
	```
	deployment:
  		staging:
    		branch: master
		    heroku:
		      appname: your-app-name
	```

	Commit and push again. Assuming you were green before, you should see your code is now
	deployed to the heroku app you created. 
