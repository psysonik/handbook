# Karma Test Runner

## Getting Started
Determine where phantomjs is installed

	$ which phantomjs
	/usr/local/bin/phantomjs
 
Update ~/.bashrc with phantomjs details and add to the top of the file:

	export PHANTOMJS_BIN=/usr/local/bin/phantomjs
 

Update the node modules in your checkout and run make to install karma within your OSX environment.

	git submodule update --recursive
	make
	
## Running Javascript Tests
The main command to run the automated Javascript test suite is as follows.

	./bin/karma start
	
This command will run the entire Javascript test suite on all browsers you have installed locally (Firefox, Chrome, PhantomJS, Opera, Safari).

There are multiple --flags that you can pass to the test runner to toggle various actions.

### Flags

#### --dev
Start test runner in dev mode, meaning it will not perform a single test run and quit, it will instead watch for file changes and automatically re-run the test suite on any changes.
#### --browsers
Pass a comma seperated list of browsers you wish to run the test suite on. Options are:
- Local Browsers
  - Chrome
  - Safari
  - Opera
  - PhantomJS
  -  Firefox
- SauceLabs Browsers

To use SauceLabs browsers, you must first have a SauceLabs Connect tunnel open to your machine. You can do this by using the SauceLabs Mac OSX application available in the App Store, or you can run the following command after you have setup a few configurations.

	cat > ~/.saucelabs.json <-EOF
	{
	    "username": "SAUCELABS USERNAME",
	    "api_key": "SAUCELABS APIKEY"
	}
	EOF
	cat > ~/.bashrc <-EOF
	export SAUCE_USERNAME="SAUCELABS USERNAME"
	export SAUCE_APIKEY="SAUCELABS APIKEY" 
	./bin/saucelauncher create-tunnel
	./karma/chrome26-windows7.js
	./karma/firefox21-windows2008.js
	./karma/ie10-windows2008.js
	./karma/ie7-windowsxp.js
	./karma/ie8-windows7.js
	./karma/ie8-windowsxp.js
	./karma/ie9-windows2008.js
	./karma/opera12-windows7.js
	./karma/saucelauncher.js
	--test
	EOF

Path to the test file you wish to run, by default this is src/ui/js/spec/index.js, but you can provide a single test file to run only that suite.

# Writing Javascript Tests
Our javascript test structure follows the convention of creating tests for javascript files in a mirrored directory structure that starts in "src/ui/js/specs/". If you have a module "src/ui/js/foo/bar/baz.js" you would create the tests for this module in "src/ui/js/specs/foo/bar/baz.js" and would add them to the test suite by adding nested describe() blocks starting at "src/ui/js/specs/index.js" and adding the appropriate files to include those nested files.

The basic structure of a test suite loader file looks like the sample below. Notice that it must return a function, and that when you require a sub test, it must call that file (since it as well should return a function). This is necessary to ensure that the describe block statements contained within the file are not execute at file load time, but instead at the place where we actually require and execute that file.

##Test Loader File Template

	define(function() {
	    return function() {
	        describe("foo", function() {
	            describe("bar", function() {
	                describe("baz", function() {
	                    require("specs/foo/bar/baz")();
	                });
	            });
	        });
	    };
	}); 
## Test Suite File Template

	define(function() {
	    var expect = require('chai').expect;
	    return function() {
	        describe('BazClass', function() {
	 
	            beforeStep(function() {
	                // Test setup code that should be run before every "it" test block
	            });
	 
	            afterStep(function() {
	                // Test teardown code that should be run after every "it" test block
	            });
	 
	            it("should meet some expectation that I have defined here.", function() {
	                 expect(1).to.equal(1);
	            });
	        });
	    };
	});
	
You can read about the matcher library Chai at [http://chaijs.com/api/bdd/])(http://chaijs.com/api/bdd/)