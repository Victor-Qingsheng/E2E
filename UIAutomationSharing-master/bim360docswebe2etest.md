# BIM 360 Docs Web UI Automation Knowledge Sharing

## 1-Automation stack
- [Nightwatch JS](http://nightwatchjs.org/)
- [Mocha](https://mochajs.org/)
- [Chai](http://chaijs.com/)
- [Jenkins](http://bim360docs-ci.ecs.ads.autodesk.com:8080/view/e2e-testing/)


## 2-A simple example
- **add.js**
```javascript
// add.js
function add(x, y) {
  return x + y;
}
module.exports = add;
```
- **add.test.js**
```javascript

// add.test.js
var add = require('./add.js');
var expect = require('chai').expect;

describe('加法函数的测试', function() {
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
});
```

```vim
$ npm install mocha
$ npm install chai
$ mocha add.test.js

  加法函数的测试
    ✓ 1 加 1 应该等于 2

  1 passing (8ms)
```
![](/images/demo1.gif)
## 3 - Chai
> Chai is a BDD(Behavior Driven Development) / TDD(Test-Driven Development) **assertion** library for node and the browser that can be delightfully paired with any javascript testing framework.

- **Should**
```javascript
chai.should();

foo.should.be.a('string');
foo.should.equal('bar');
foo.should.have.lengthOf(3);
tea.should.have.property('flavors')
  .with.lengthOf(3);
```

- **Expect**
```javascript
var expect = chai.expect;

expect(foo).to.be.a('string');
expect(foo).to.equal('bar');
expect(foo).to.have.lengthOf(3);
expect(tea).to.have.property('flavors')
  .with.lengthOf(3);
```
- **assert**
```javascript
var assert = chai.assert;

assert.typeOf(foo, 'string');
assert.equal(foo, 'bar');
assert.lengthOf(foo, 3)
assert.property(tea, 'flavors');
assert.lengthOf(tea.flavors, 3);
```
further:http://chaijs.com/api/
## 4 - mocha
> Mocha is a feature-rich JavaScript test framework running on Node.js and in the browser, making asynchronous testing simple and fun.

### 4.1 - How to write test case
- **require**
```javascript
var add = require('./add.js');
var expect = require('chai').expect;
```
- **describe**:test suite(测试套件)
- **it**:test case (测试用例)
```javascript
describe('test module', function(){
  describe('test function add',function(){
    it('should correctly add.',function({
      //assertion here
    }));
  });

  describe('test function minute',function(){
    it('should correctly minute.',function({
      //assertion here
    }));

    it('should correctly minute.',function({
      //assertion here
    }));
  });
});
```

- **Asynchronous code**
  - Adding a callback (usually named done) to it(),so mocha will know test is complete
  ```javascript
  describe('User', function() {
    describe('#save()', function() {
      it('should save without error', function(done) {
        var user = new User('Luna');
        user.save(done);
      });
    });
  });
  ```
  - **Return a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)**
  ```javascript
  beforeEach(function() {
  return db.clear()
    .then(function() {
      return db.save([tobi, loki, jane]);
    });
  });

  describe('#find()', function() {
    it('respond with matching records', function() {
      return db.find({ type: 'User' }).should.eventually.have.length(3);
    });
  });
  ```
  - **Using [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)**
  ```javascript
  beforeEach(async function() {
    await db.clear();
    await db.save([tobi, loki, jane]);
  });

  describe('#find()', function() {
    it('responds with matching records', async function() {
      const users = await db.find({ type: 'User' });
      users.should.have.length(3);
    });
  });
  ```

- synchronous code
```javascript
describe('Array', function() {
  describe('#indexOf()', function() {
    it('should return -1 when the value is not present', function() {
      [1,2,3].indexOf(5).should.equal(-1);
      [1,2,3].indexOf(0).should.equal(-1);
    });

    it(.....);
  });
});
```

- arrow functions (“lambdas”) //will losing mocha context
[Do ES6 Arrow Functions Really Solve “this” In JavaScript?](https://derickbailey.com/2015/09/28/do-es6-arrow-functions-really-solve-this-in-javascript/)
```javascript
describe('my suite', () => {
  it('my test', () => {
    // should set the timeout of this test to 1000 ms; instead will fail
    this.timeout(1000);
    assert.ok(true);
  });
});
```

- HOOKS --ASYNCHRONOUS
```javascript
describe('hooks', function() {

  before(function() {
    // runs before all tests in this block
  });

  after(function() {
    // runs after all tests in this block
  });

  beforeEach(function() {
    // runs before each test in this block
  });

  afterEach(function() {
    // runs after each test in this block
  });

  // test cases
});
```

- Management of test case//or using parameters
  - exclusive tests .only()
  ```javascript
  describe('Array', function() {
    describe.only('#indexOf()', function() {
      it.only('should return -1 unless present',       function() {
        // this test will be run
      });

      it('should return the index when present',   function() {
        // this test will not be run
      });
    });
  });
  ```
  - inclusive tests
  ```javascript
  describe('Array', function() {
    describe('#indexOf()', function() {
      it.skip('should return -1 unless present', function() {
        // this test will not be run
      });

      it('should return the index when present',   function() {
        // this test will be run
      });
    });
  });
  ```
- Retry tests
> Sometime we need to retry failed tests up to a certain number of times for e2e tests.
```javascript
describe('retries', function() {
  // Retry all tests in this suite up to 4 times
  this.retries(4);

  beforeEach(function () {
    browser.get('http://www.yahoo.com');
  });

  it('should succeed on the 3rd try', function () {
    // Specify this test to only retry up to 2 times
    this.retries(2);
    expect($('.foo').isDisplayed()).to.eventually.be.true;
  });
});
```

- Dynamically generating tests
```javascript
var assert = require('chai').assert;

function add() {
  return Array.prototype.slice.call(arguments).reduce(function(prev, curr) {
    return prev + curr;
  }, 0);
}

describe('add()', function() {
  var tests = [
    {args: [1, 2],       expected: 3},
    {args: [1, 2, 3],    expected: 6},
    {args: [1, 2, 3, 4], expected: 10}
  ];

  tests.forEach(function(test) {
    it('correctly adds ' + test.args.length + ' args', function() {
      var res = add.apply(null, test.args);
      assert.equal(res, test.expected);
    });
  });
});
```

- Test duration
```javascript
var expect = require('chai').expect;

describe('a suite of tests', function() {
  this.timeout(500);

  it('should take less than 500ms', function(done){
    setTimeout(done, 300);
  });

  it('should take less than 500ms as well', function(done){
    this.timeout(500);
    setTimeout(done, 250);
  });
})
```

### 4.2 - How to run case
- $ mocha add.test.js
![](/images/demo001.gif)
- $ mocha filename1.test.js filename2.test.js
- $ mocha
- $ mocha --recursive
![](/images/recursive.gif)
**Using mochawesome**
- $ npm install mochawesome
- $ mocha --recursive --reporter mochawesome
![](/images/mochawesome.gif)
![](/images/demo2reporter.png)

## 5 - Nightwatch.js -Browser Automation
- [W3C WebDriver API](https://www.w3.org/TR/webdriver/)
- chai
- mocha
- node.js


### 5-1 Overview of WebDriver
> **WebDrive**r is a general purpose library for automating web browsers. It was started as part of the **Selenium** project, which is a very popular and comprehensive set of tools for browser automation, initially written for Java but now with support for most programming languages.
>
>**Nightwatch** uses the WebDriver API to perform the browser automation related tasks, like opening windows and clicking links for instance.
>
>WebDriver is now a W3C specification, which aims to standardize browser automation. <font color=#23c8ec>WebDriver is a remote control interface that enables introspection and control of user agents. It provides a platform and a restful HTTP api as a way for web browsers to be remotely controlled.</font>

### 5-2 Theory of Operation
- **workflow**
> Nightwatch works by communicating over a restful HTTP api with a WebDriver server (typically the Selenium server). The restful API protocol is defined by the W3C WebDriver API. See below for an example workflow for browser initialization.
>
![](/images/operation.png)

> 1. Request to locate an element given a CSS selector (or Xpath expression)
> 2.  perform the actual command/assertion on the given element.

### 5.3 How to use Nightwatch
- **1Install Nightwatch**
```
$ git clone&&npm install [-g] nightwatch
```
- **2Selenium Server Setup**
  - Download selenium-server-standalone-{VERSION}.jar
  - java -jar selenium-server-standalone-{VERSION}.jar
- **3Configuration**
  - nightwatch.conf.js
  ```
  {
    "src_folders" : ["tests"],
    "output_folder" : "reports",
    "custom_commands_path" : "",
    "custom_assertions_path" : "",
    "page_objects_path" : "",
    "globals_path" : "",

    "selenium" : {
      "start_process" : false,
      "server_path" : "",
      "log_path" : "",
      "port" : 4444,
      "cli_args" : {
        "webdriver.chrome.driver" : "",
        "webdriver.gecko.driver" : "",
        "webdriver.edge.driver" : ""
      }
    },

    "test_settings" : {
      "default" : {
        "launch_url" : "http://localhost",
        "selenium_port"  : 4444,
        "selenium_host"  : "localhost",
        "silent": true,
        "screenshots" : {
          "enabled" : false,
          "path" : ""
        },
        "desiredCapabilities": {
          "browserName": "firefox",
          "marionette": true
        }
      },

      "chrome" : {
        "desiredCapabilities": {
          "browserName": "chrome"
        }
      },

      "edge" : {
        "desiredCapabilities": {
          "browserName": "MicrosoftEdge"
        }
      }
    }
  }
  ```

- 4 Use CSS & Xpath selectors to locate and verify elements on the page or execute commands
```
:class             => 'class name',
:class_name        => 'class name',
:css               => 'css selector',
:id                => 'id',
:link              => 'link text',
:link_text         => 'link text',
:name              => 'name',
:partial_link_text => 'partial link text',
:tag_name          => 'tag name',
```
![](/images/csselement.png)
```javascript
this.waitForElementVisible('@rfiTabBtnDocs', this.api.globals.waitForConditionTimeout).click('@rfiTabBtnDocs');
```
- 5 more about nightwatch
http://nightwatchjs.org/guide#using-nightwatch


- **6Example**
  - [test.js](./Demo/learn-nightwatch/examples/tests/google/googleDemoTest.js)
  - [nightwatch.conf.BASIC.js](./Demo/learn-nightwatch//nightwatch.conf.BASIC.js)

![](/images/nightwatchdemo2.gif)

## 6 - BIM360 Docs Web UI Automation
- framework
```
├── globalsModule.js
├── helpers
│   ├── commonFunctions.js
│   ├── pageObjects.js
│   └── utils.js
├── pageObjects
│   ├── common
│   │   ├── common.js
│   │   ├── documentNavigation.js
│   │   ├── login.js
│   │   ├── matrixHeader.js
│   │   ├── projectNavigation.js
│   │   └── siteNavigation.js
│   ├── history
│   │   └── historyDocumentLevel.js
│   ├── issues
│   │   ├── issuesDocumentLevel.js
│   │   └── issuesProjectLevel.js
│   ├── markup
│   │   └── markupDocumentLevel.js
│   ├── permissions
│   │   └── permissionPageLevel.js
│   ├── rfis
│   │   ├── rfisDocumentLevel.js
│   │   └── rfisProjectLevel.js
│   └── titleblock
│       └── titleBlockPageLevel.js
├── testData
│   ├── 1floor-v1.dwg
│   ├── 2D__3D__BOM.dwf
├── testReport
│   ├── reports
│   └── screenshots
└── tests
    ├── attributes
    │   └── plans
    │       └── AddAttributeOnPlansSmokeTest.js
    ├── documents
    │   ├── plans
    │   │   └── documents.js
    │   └── project_files
    │       └── documents.yml
    ├── download
    │   ├── CheckFileNameSmokeTest.js
    │   └── download.js
    ├── folders
    │   ├── plans.js
    │   └── project_files.js
    ├── issues
    │   └── smoke
    │       ├── document
    │       │   └── issuesDocumentSmokeTest.js
    │       └── project
    │           ├── issuesProjectFiltering.js
    │           └── issuesProjectSorting.js
    ├── login
    │   └── login_test.js
    ├── permissions
    │   └── plans
    │       └── PermissionsOnPlansSmokeTest.js
    ├── rfis
    │   ├── smoke
    │   │   ├── document
    │   │   │   └── rfisDocumentSmokeTest.js
    │   │   └── project
    │   │       └── rfiProjectFiltering.js
    │   └── workflows
    │       ├── workflowA.js
    │       └── workflowB.js
    ├── smoke
    │   └── smoke_test.js
    ├── subscribefolder
    │   ├── plans
    │   │   └── SubscribePlansFolderSmokeTest.js
    │   └── projectfiles
    │       └── SubscribeProjectFilesSmokeTest.js
    ├── titleBlock
    │   └── plans
    │       └── TitleBlockOnPlansSmokeTest.js
    ├── upload
    │   ├── plans
    │   │   └── upload_to_plans.js
    │   └── project_files
    │       └── upload_to_project_files.js
    ├── versionhistory
    │   └── plans
    │       └── VersionHistoryOnPlansSmokeTest.js
    └── viewer
        ├── plans
        │   ├── 2DViewSmokeTest.js
        │   └── 3DViewSmokeTest.js
        └── project_files
            ├── 2DsheetSmokeTest.js
            └── 3DsheetSmokeTest.js
```

- How to start
1. git clone https://git.autodesk.com/BIM360/BIM360DM-web-ui-automation/
2. npm install
- Running tests
1. Install globally the ntl package by running npm install -g ntl.
2. Then run the test scripts with ntl or by running npm run 'script_name'

## 7 - Jenkins
http://bim360docs-ci.ecs.ads.autodesk.com:8080/
