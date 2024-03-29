## Unit Testing in TypeScript

This guide will show you how to setup a TypeScript project to set up unit testing.


### Introduction

There are a variety of tools that can be used for Unit Testing in TypeScript or Javascript.  These are the ones used in this example.

#### ts-node
[ts-node](https://typestrong.org/ts-node/docs/) is a JIT compiler that allows your test code to directly execute the TypeScript which allows the code coverage to be tracked at the TypeScript level instead of the complile Javascript

#### Mocha
[Mocha](https://mochajs.org/) is a feature-rich test framework which will execute the tests.

#### Chai
[Chai](https://www.chaijs.com/) is an assertion library which iss BDD or TDD style asserts.  It allows three classes of assertions where the same assertions are shown using it below.


##### Should
```Javascript
foo.should.equal(‘bar’);
foo.should.have.lengthOf(5);
```

##### Expect
```Javascript
expect(foo).to.equal(‘bar’);
expect(foo).to.have.lengthOf(3);
```

##### Assert

```Javascript
assert.equal(foo, ‘bar’);
assert.lengthOf(foo, 3);
```

#### Nock
[Nock](https://github.com/nock/nock) is a mocking framework that allows you to mock HTTP requests to calls to an external service.  You can mock the response to a known response allowing the testing to focus on your code. 

#### Sinon
[Sinon](https://sinonjs.org/) is a another mocking framework that allows you to create spies, stubs and mocks to provide a known response when testing your code. 

#### Nyc/Istanbul  
[Istanbul](https://istanbul.js.org/) is a test coverage tool that will analyze and report on the test coverage provided by your test cases.<br>
[Nyc](https://github.com/istanbuljs/nyc) is the command line client that works with Istanbul


### Set up your project
To set up your project, run the following to install the dependencies and update package.json

```
$ npm install mocha chai ts-node nyc @types/mocha
```

If you have not done it already, create a test folder at the project root for the testing files.

In that folder create a file **_test/tsconfig.json_** to configure TypeScript for testing containing the following.  The will set configuration for things executed from the test folder to use ts-node to transpile instead of compile and execute as Javascript.

```Javascript
{
    "extends": "../tsconfig.json",
    "ts-node": {
      "transpileOnly": true,
      "files": false
    },
    "compilerOptions": {
      "baseUrl": "./",
      "module": "commonjs",
      "experimentalDecorators": true,
      "strictPropertyInitialization": false,
      "isolatedModules": false,
      "strict": false,
      "noImplicitAny": false,
      "typeRoots" : [
        "../node_modules/@types"
      ]
    },
    "exclude": [
      "../node_modules"
    ],
    "include": [
      "./**/*.ts"
    ]
  }
```
  
We are also going to use dotenv so we can move configuration that might be in a ConfigMap to a .env file.  Create a file **_test/.env_** to contain the environment values you will used when executing a test in a variable=value format.  Note these do not have to be exactly like the ones on a test or production environment but will make to what is expected with tests


```Javascript
testEndpoint=https://localhost/test
testApiKeyId=testApiKeyIdValue  
```

Now to turn on this ts-node and dotenv configuration, create a file **_./.mocharc.json_** in the project root and add the following.

```Javascript
{
    "require": "./register.js",
    "reporter": "dot"
}
```

Next, create a file **_./register.js_** at the project root and add the following.  This will use our env file with dotenv and call our ts-node configuration when we are testing.

```Javascript
/**
 * Overrides the tsconfig used for the app.
 * In the test environment we need some tweaks.
 */

// Read .env for access to process.env
 const dotenv = require('dotenv')
 dotenv.config({path: './test/.env'});

 // Enable tsNode
 const tsNode = require('ts-node');
 const testTSConfig = require('./test/tsconfig.json');
 
 
 tsNode.register({
   project: './test/tsconfig.json'
 });
```


To set up the code coverage configuration, add the following to the file **_./.nyrc.json_** in the project root

```Javascript
{
    "extends": "@istanbuljs/nyc-config-typescript",
    "all": true,
    "check-coverage": true,
    "include": [
      "src/**/*.ts"
    ],
    "exclude": [
      "node_modules/"
    ],
    "extension": [
      ".ts"
    ],
    "reporter": [
      "text-summary",
      "html"
    ],
    "report-dir": "./coverage"
}
```

You will want to be sure all of the source code for which coverage should be measured is in the include array.

And be sure that any code that should be excluded (in this case node_modules) is in the exclude array.


Add the following to the scripts array in **./_package.json_** in the project root to provide a command to test
```Javascript



"test": "nyc --all --reporter=lcov --reporter=text --reporter=text-summary ./node_modules/.bin/_mocha --reporter=spec 'test/**/*.test.ts'",
"posttest": "nyc report --reporter=lcov"
```

### Create a Simple Test
Now it is time to create test cases.

There are two styles that can be used for test case creation.

**Testdeck** – This is a wrapper for Mocha which allows more object-oriented behaviour.  But there is not a lot of documentation for it so use with caution.  The documentation for [testdeck](https://testdeck.org/) can be found here. This is used in https://github.ibm.com/ibm-api-marketplace/playground-workspaces/blob/master/test/pool.calculator.test.ts.  
  
With this you have a test class (with an @suite assertion) with test functions (with an @test assertion).  There are special functions, like before() which is executed before each test that can be used.

**Mocha** – This is more standard mocha way of testing and will be explored more here.  The documentation can be found [here](https://mochajs.org/#getting-started)

For our first example we will test a class that reads environment information and provides information about it.

First, update **_test/.env_** with the following lines for the config that we will test.  Note how we have simplified the data to be generic for our test case.

```Javascript
devfiles=devfile1,devfile2
devfile_devfile1={"devfile": "actual-devfile1","weight": "5","default": "true"}
devfile_devfile2={"devfile": "actual-devfile2","weight": "2"}
```

Create a file for the test case.   It is probably best to name it related to the file that it is testing.  

Create **_test/devfile.controller.test.ts_** and add the describe function to it

```Javascript
import { assert, expect, should } from 'chai';
import { utils } from 'mocha';
import nock from 'nock';
import { DevfileController } from '../src/controller/devfile.controller';
import { DevfileConfig } from '../src/model/devfile.config';
const yaml = require('js-yaml');

let devfileController: DevfileController = DevfileController.getInstance();

describe("Testing DevfileController", () => {

});
```

Now let’s add a test inside it.

Add the following test.  The function “it” is taken from BDD and literally means it with the parameter describing the test.

This test verifies one of the environment variables and the isDefault property that is set with it.

```Javascript
describe("Testing DevfileController", () => {

  /**
   * Tests getDevfile()
   */
  it('getDevfile', () => {
    let res: DevfileConfig = devfileController.getDevFileConfig("devfile_devfile1");
    assert.equal(res.devfilename, "actual-devfile1");
    assert.equal(res.isDefault, true);
  });

});
```

Save this and now you can test it.

To run all of the test you can run **_npm test_** which will run the test but also show the test coverage on the files testing including the lines not tested.  We will show more on that later.

```
$ npm test

> playground-monitoring@0.0.0 test
> nyc --all --reporter=lcov --reporter=text --reporter=text-summary ./node_modules/.bin/_mocha --reporter=spec 'test/**/*.test.ts'



  Testing DevfileController
    ✔ getDevfile


  1 passing (4ms)

--------------------------------------|---------|----------|---------|---------|-------------------
File                                  | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
--------------------------------------|---------|----------|---------|---------|-------------------
All files                             |   54.63 |    47.36 |    25.8 |   54.63 |                   
 playground-workspaces                |     100 |      100 |     100 |     100 |                   
  register.js                         |     100 |      100 |     100 |     100 |                   
 playground-workspaces/src/controller |   69.04 |       50 |   27.27 |   69.04 |                   
  devfile.controller.ts               |   69.04 |       50 |   27.27 |   69.04 | 58-92             
 playground-workspaces/src/model      |   72.72 |       40 |      50 |   72.72 |                   
  devfile.config.ts                   |   72.72 |       40 |      50 |   72.72 | 19-22             
 playground-workspaces/src/service    |   63.63 |      100 |      50 |   63.63 |                   
  git.service.ts                      |   63.63 |      100 |      50 |   63.63 | 19-25             
 playground-workspaces/src/utils      |   14.28 |      100 |   14.28 |   14.28 |                   
  https.ts                            |   14.28 |      100 |   14.28 |   14.28 | 15-89             
--------------------------------------|---------|----------|---------|---------|-------------------

=============================== Coverage summary ===============================
Statements   : 54.63% ( 53/97 )
Branches     : 47.36% ( 9/19 )
Functions    : 25.8% ( 8/31 )
Lines        : 54.63% ( 53/97 )
================================================================================

> playground-monitoring@0.0.0 posttest
> nyc report --reporter=lcov
```


Or to run a specific test, you can run.   The option –reporter=spec shows more details on each test “it” run.
```
$ ./node_modules/.bin/_mocha  --reporter=spec test/devfile.controller.test.ts

  Testing DevfileController
    ✔ getDevfile


  1 passing (23ms)
```
You can now add more tests

```Javascript
  /**
   * Tests getDefaultDevfile()
   */
  it('getDefaultDevfile', () => {
    let res: string = devfileController.getDefaultDevFile();
    assert.equal(res, "devfile1");
  });
  /**
   * Tests getWeight()
   */
  it('getWeight', () => {
    let res = devfileController.getWeight("devfile_devfile1");
    assert.equal(res, 5);
  });

  /**
   * Tests getWeightTotal()
   */
  it('getWeightTotal', () => {
    let res = devfileController.getWeightTotal();
    assert.equal(res, 7);
  });
```

### Ignoring Code in Coverage Reports
In some cases, it does not make sense to unit test some lines of code, though this should be used cautiously.  In this example, there is a function that is there if we want to log more information but the developer decides it does not need to be tested.  By adding this “special” comment 

/* istanbul ignore next */

nyc will ignore these line in code coverage

```Javascript
  public async logDevFiles() {
      /* istanbul ignore next */
      this.devfiles.forEach(async (value: DevfileConfig, key: string) => {
          this.logger.info(key);
          this.logger.info((await value.getDevFile()).toString());
      });
  }
```

### Create a mock for external dependencies using Nock

One of the functions we want to test eventually makes an external API call.  In unit testing you do not want to call anything external for a variety of reasons including
•	Is the service up?
•	Extra testing load on the service
•	What if the data changes

The following will show you how to mock (using nock) the results of the API call.

Here is the function we want to test

```Javascript
    public async overrideMetaName(devfile: string, metaname: string) :
  Promise<WorkspaceDevfile> {
        let devFileStr: string = await this.gitService.getDevFileByName(this.devfiles.get(devfile).devfilename);
        let devFileObj :any = yaml.safeLoad(devFileStr);
        if (typeof devFileObj === "object" && typeof devFileObj.metadata.name === 'string') {
            devFileObj.metadata.name = metaname;
        } else {
            /* istanbul ignore next */
            this.logger.error("Parsing error");
        }
        //return devFileObj;
        return JSON.parse(JSON.stringify(devFileObj));
    }
```

But, _this.gitService.getDevFileByName(string)_ is an external call.

First, we create a file with the response we want back from that call.  Often (and maybe normally), the response is json and a .json file is appropriate, but in this case it is a yaml file.  Create the following file at **_test/mock-responses/gitdevfile.yaml_**.  Note we are going to put are going to use **_test/mock-responses_** for these responses

```yaml
apiVersion: 1.0.0
metadata:
  name: sample-app
attributes:
  persistVolumes: 'false'
```

Now let’s dig into the test that will use it

```Javascript
  /**
   * Tests overrideMetaName()
   */
  it('overrideMetaName', async () => {
    const gitURL = "https://raw.githubusercontent.com/IBM/Developer-Playground/master/devfiles"
    nock(gitURL)
      .get("/actual-devfile1.yaml")
      .replyWithFile(200, __dirname + '/mock-responses/gitdevfile.yaml');
    let res = await devfileController.overrideMetaName("devfile_devfile1", "newmetaname");
    let metaname;
    if (typeof res === "object" && typeof res.metadata.name === 'string') {
      metaname = res.metadata.name;
    }
    assert.equal(metaname, "newmetaname");
  });
```

The API call that we want to mock is a GET request to https://raw.githubusercontent.com/IBM/Developer-Playground/master/devfiles/actual-devfile1.yaml

And we want the mocked file above to be the response.

This block of code creates a mock response so that when the gitURL is called with the GET request for */actual-devfile1.yaml*, that a 200 response will sent along with the content in  **_/mock-responses/gitdevfile.yaml_**.

```Javascript
  nock(gitURL)
    .get("/actual-devfile1.yaml")
    .replyWithFile(200, __dirname + '/mock-responses/gitdevfile.yaml');
```
In this function the *metadata.name* value (*sample-app* in the mock response) is overridden with a variable passed to the function, but the mocking does allow us to test the parsing of the response data

Note that nock will ONLY override the request the first time this request is made.  Another call to it will actually call the external service.  In this case we defined it in the test case because we were only calling it once

If we have a repeated call that we always mock the response we can include a **beforeEach** function in the test.

Here is an example where an authentication token is mocked for repeated authorization calls in the test that would follow.

```Javascript
  beforeEach(function () {
      // Do this everytime since the same result is needed
      const mockResponse_keycloakToken = require('./mock-responses/keycloak-token.json');
      nock(keyCloakUrl)
          .post(userTokenEndpoint)
          .reply(200, mockResponse_keycloakToken);
      this.timeout(10000);
  })
```

### Create a stub for external dependencies using Sinon

Sometimes you cannot mock at the HTTP level.  A good example is the SDK used to interact with Cloud Object Storage.  The communications with Object Storage is encapsulated in the SDK and not easily mocked.   Sinon allows you to create a stub for a function call so that you can intercept it before the SDK and return a mocked response.

In this case we want to mock the response to this function in our code in the CoS class

```Javascript
  public async readItem(bucketName: string, itemName: string)
```

This code sets up the "stub" to respond whenever that function is called directly or indirectly in a test.  Note that the mocked response returned will be the contents of *__./mock-responses/cosdevfile.yaml__*.  In this case the item in Object Storage is a string that just happens to be yaml, but it is treated as a string and could be JSON or any other string.

Note  `.stub(<object>,<function>` which connects the stub to the function.  And the function defined in `.callsFake(<function() {});>)` provides what will be done in the stub

```Javascript
  let cos = CoS.getInstance();
  let sandbox;

  before(() => {
    /* Use signon fake instead of nock to mock the readItem function as the COS SDK makes 
    *  multiple calls which make it very to use nock.  This overrides any call to the readItem method.
    */
    sandbox = sinon.createSandbox();
    sandbox
      .stub(cos, "readItem")
      .callsFake(function  fakeReadItem (bucketName: string, itemName: string) {
        return fs.readFileSync(__dirname + '/mock-responses/cosdevfile.yaml');
      });

  });
```

In our test, it is the call to `devfileService.getDevFile();` that ultimately calls the stub and we can make assertions based on known values.

```Javascript
  /**
   * Tests DevFileService() with cos
  */
  it('DevFileService() with cos', async () => {


    // Using true to force a new instance to clear anything set in previous test.
    let devfileService: DevfileService = DevfileService.getInstance(true);
    assert.equal(devfileService.getServiceProvider(),"cos");
    let devFileObj :any = await devfileService.getDevFile();
    assert.equal(typeof devFileObj,"object");
    assert.equal(typeof devFileObj.metadata.name,'string');
    if (typeof devFileObj === "object" && typeof devFileObj.metadata.name === 'string') {
      assert.equal(devFileObj.metadata.name,"sample-app-cos");
    }
  });
```

At the end of the test, you must use a restore to cancel out the stub

```Javascript
  after(() => {
    // Restore sinon fake
    sandbox.restore();
  });
```

### Code Coverage reports

Remember that we defined the npm test script to run nyc in **_./package.json_**

And we defined the options in **_./.nyrc.json_**

With this when you run npm test, it will update the following directories below.

**_./.nyc-output_**  - process and coverage information

**_./coverage_** – location defined in the options for for coverage report which you be viewed in a browser with file **_./coverage/index.html_**   In this we can drill down to see the coverage we have on the file we were testing


Also note that you should add these directories to .gitignore as you do not want to commit these coverages.  Instead anything that uses the coverage in the DevOps process should run the test each time you want to evaluate. 

