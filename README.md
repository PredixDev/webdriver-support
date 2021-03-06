Webdriver Support
=================

Support for running automated __functional__ tests with WebdriverJs + Mocha + Chai.

## Contains

* Binaries needed for testing (e.g. Mocha, selenium-server-standalone.jar)
* Default config for webdriverjs+Mocha+Chai
* A grunt task for configuration of runs using this package

`peerDependencies` for use in tests include:
* chai: http://chaijs.com/
* chai-as-promised: http://chaijs.com/plugins/chai-as-promised

## Of note

* UI / functional tests are Mocha + Chai + WebdriverJs.  Mocha defines and runs tests, Chai is an assertion library, and WebdriverJs drives the simulated user interaction.
    * Webdriver tests applications / test fixtures __remotely__ from the __point of view of the user__, not the code.  It simulates the user, and uses a plain vanilla "application under test", the application or page that you can pull up with any http server and inspect.  http-server is a npm dependency in package.json so you can run http-server at any time and access your ui test fixture(s) the exact same way webdriver does.
    * Upon the start of a test Webdriver-support does the following:
        * Starts webdriver server
        * Starts `PhantomJsDriver` or `ChromeDriver` or `FirefoxDriver` per the test configuration
        * Starts a nodeJs http server to run the "application under test" / test fixture
        * Let's Mocha do the testing, Mocha will tell Webdriver to do things in the "application under test"
    * Webdriver is asynchronous and almost every command returns a promise object. The tests need to deal with promises / asynchronous execution.  There are two options to do this...see `/examples` for examples of each.
        * Use chai-as-promised to assert across promises (preferred), and/or,
        * Put assertions inside `then` statements (standard promise stuff) and Mocha's `done` functionality to tell Mocha when a promise is resolved.
    * Upon failure of a test, WebdriverJs takes a DOM dump and a screenshot and logs those + a stacktrace in a TAP-compatible (Test Anything Protocol) format for use with Mocha's TAP reporter.
* Default to use phantomjs but chrome and firefox are available, too
* Looking for unit testing (testing from the point of view of the __code__ rather than from the point of view of the __user__? Unit tests using Mocha + Chai + Sinon + Karma are supported by the https://github.com/PredixDev/karma-support library

## Rationale

### General

There are two types of testing in the presentation layer: Unit, Functional/UI (including "visual assertion", e.g. verification of actual rendering by comparing DOM or screenshots).  For all three, there's a lot of configuration to accomplish before a developer can get up and running with tests.

This project abstracts the test boilerplate of functional/ui testing so the project itself can include just the tests.  "Visual assertion" within tests supported by this package can be done using https://github.com/PredixDev/vissert . Unit testing is supported separately in https://github.com/PredixDev/karma-support

For all types of tests, we need:

* Speed, can be divided into suites (smoke test vs. full regression test).  Unit tests, in particular, must be very fast
* Multi-browser, incl. headless (PhantomJS)
* Clear failures with stack traces
* Tooling for breakpoints in IDE (or other)
* Enable independent run of failures
* Enable CI trending of failures over time.
* BDD vocabulary option
* Tie into other tools, e.g. Istanbul (code coverage), Protractor (AngularJs-optimized wrapper around WebdriverJs).

### Functional / UI testing

In functional/UI testing, the main additional requirements are:

 * Uses "plain" HTML Fixtures (can load up same HTML used for the test outside the test framework to debug) -- enables work to be spun up on any HTTP server for review by design, product management and other stakeholders remotely and with a minimum of tooling
 * Useful debug information upon failure, e.g. Screenshots / DOM dumps / stack traces -- enables debugging on remote environments such as CI which might have unique failures not seen locally.
 * Capability to decouple DOM targeting from tests so tests are less brittle (see http://martinfowler.com/bliki/PageObject.html)

### Unit testing

Unit tests using Mocha + Chai + Sinon + Karma are supported by the https://github.com/PredixDev/karma-support library

## Usage

1. Include this package as a NPM dependency in your project's package.json: `"webdriver-support": "git+https://github.com/PredixDev/webdriver-support.git#[version tag]"`
2. Run NPM install
3. Scaffold out your project's tests in `test/ui` of your project using the sample files in /examples (note: A few Yeoman Generators will do this for you: `generator-dx-component` for GE Design System Components, and `generator-card` for Predix Reader Cards)
4. Add the relevant Grunt tasks and config to your project's Gruntfile.js (see note above about Yeoman Generators)
5. Run 'grunt webdriver'

## Writing Tests

See the documentation of Mocha (http://mochajs.org/ with links to Sinon and Chai) and WebdriverJs (https://code.google.com/p/selenium/wiki/WebDriverJs)

## CI Integration

Mocha controls the report output of tests via its reporters.  The TAP ("Test Anything Protocol") reporter is useful, and this package will format a DOM Dump, Stack Trace and Screenshot for TAP upon failure by default when the TAP reporter is used.

## Future plans

* Demonstrate (enforce?) usage of the PageObject pattern for WebdriverJs

## Windows Notes

* Even if Java is available via command line, it still needs to be in the system path/environmental variable
* http-server install may fail, I think it needs to be installed both globally (npm install -g http-server) as well as inside the node_modules for webdriver-support (PLEASE VERIFY!)
* Observed that on some machines, the PhantomJS exe is added to node_modules, but others not. Windows depends on the exe existing at "node_modules/webdriver-support/node_modules/phantomjs/lib/phantom/phantomjs.exe". If the exe is not there, please download from the official project and add it.
