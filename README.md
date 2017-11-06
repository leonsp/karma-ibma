# Web Accessibility Workshop - CASCON 2017

Instructions how to use `@ibma/karma-ibma` for Accessibility checking on web Apps built using Angular.

## Pre-requisites:
- NodeJS
- NPM

## Setup

Creating and getting to know Angular Web Application.

### Step 1. Install `@angular/cli` globally
1. Run command `npm install -g @angular/cli`

### Step 2. Create new Angular Web Application
1. Run command `ng new my-app`
2. Run command `cd my-app` to change directory into your app
3. Refer to [Angular Getting Started](https://angular.io/guide/quickstart) for more details.

### Step 3. Launching Application
1. Run command `ng serve --open`

### Step 4. Running unit/e2e tests
1. Run command `ng test` for unit tests
1. Run command `ng e2e` for e2e tests

## How to integrate `@ibma/karma-ibma` into an Angular Application

Now we will integrate accessibility testing into the Angular Application

### Install `@ibma/karma-ibma`

1. Run command `npminstall https://aat.mybluemix.net:/dist/karma-ibma.tgz--save-dev`

### Configure Karma plugins

If you are using the plugins array in the Karma configuration file (`karma.conf.js`), you will need to add this to the
plugins array to load the `@ibma/karma-ibma` plugin.

```js
// karma.conf.js
module.exports = function(config) {
  config.set({

    // Plugins for karma to load
    plugins: [
        require('@ibma/karma-ibma') // ONLY NEED to add this one line
    ]

    // ...
  });
};
```

### Configure Karma `framework` and `reporters`

Add `AAT` to the `framework`, `reporters` array in the Karma configuration file (`karma.conf.js`). This will load the `@ibma/karma-ibma` plugin.

```js
// karma.conf.js
module.exports = function(config) {
  config.set({

    // Frameworks to use to run the tests that we define
    frameworks: ['jasmine', 'AAT'], // ONLY NEED TO ADD "AAT" 

    // Test results reporter to use
    reporters: ['progress', 'AAT'], // ONLY NEED TO ADD "AAT" 

    // ...
  });
};
```

### Configure `@ibma/karma-ibma` plugin

Configuring the `@ibma/karma-ibma` plugin involves constructing a `.aat.yml` file in the project root. This file, will contain all of the configuration
options for `@ibma/karma-ibma`. This is the structure of the `.aat.yml` file:

```yml
# required - Provide the authentication token available at http://ibm.biz/a11yToolsDashboard
authToken: 00000000-0000-0000-0000-000000000000/00000000-0000-0000-0000-000000000000

# optional - Specify one or many policies to scan.
# i.e. For one policy use policies: IBM_Accessibility_2017_02
# i.e. Multiple policies: IBM_Accessibility_2017_02,IBM_Accessibility_BETA or refer to below as a list
# Default: null (all policies)
# Refer to README.md FAQ section on details to get the policy ID.
policies:
    - IBM_Accessibility_2017_02

# optional - Specify one or many violation levels on which to fail the test
#            i.e. If specified violation then the testcase will only fail if
#                 a violation is found during the scan.
# i.e. failLevels: violation
# i.e. failLevels: violation,potential violation or refer to below as a list
# Default: violation, potentialviolation
failLevels:
    - violation
    - potentialviolation

# optional - Specify one or many violation levels which should be reported
#            i.e. If specified violation then in the report it would only contain
#                 results which are level of violation.
# i.e. reportLevels: violation
# i.e. reportLevels: violation,potentialviolation or refer to below as a list
# Default: violation, potentialviolation, recommendation, potentialrecommendation, manual
reportLevels:
    - violation
    - potentialviolation
    - recommendation
    - potentialrecommendation
    - manual

# Optional - Which type should the results be outputted to
#   outputFormat: json
# Default: json
outputFormat:
    - json

# Optional - Specify labels that you would like associated to your scan
#
# i.e.
#   label: Firefox,master,V12,Linux
#   label:
#       - Firefox
#       - master
#       - V12
#       - Linux
# Default: N/A
label:
    - master

# optional - Where the scan results should be saved.
# Default: results
outputFolder: results

# optional - Should Hidden content be scanned
# true --> Yes scan hidden content
# false --> Don't scan hidden content
# Default: false
checkHiddenContent: false
```

### Write accessibility testcases

Add a new testcase to `src/app/app.component.spec.ts`.

1. Declare `AAT` by adding the following:

```typescript
declare var AAT: any;
```

2. Add a new testcases to scan for accessibility

```typescript
describe('AppComponent', () => {
  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [
        AppComponent
      ],
    }).compileComponents();
  }));
  it('accessibility testing should pass on main APP', ((done) => {
    const fixture = TestBed.createComponent(AppComponent);
    fixture.detectChanges();
    const compiled = fixture.debugElement.nativeElement;

    // Perform the accessibility scan using the AAT.getCompliance API
    AAT.getCompliance(compiled, 'Main APP', function (results) {

      // Call the AAT.assertCompliance API which is used to compare the results with baseline object if we can find one that
      // matches the same label which was provided.
      const returnCode = AAT.assertCompliance(results);

      // In the case that the violationData is not defined then trigger an error right away.
      expect(returnCode).toBe(0, 'Scanning Main APP failed.' + JSON.stringify(results));

      // Mark the testcases as done, when using jasmine as the test framework.
      done();
    });
  }));
});
```
### Run testcases
1. Run command `npm test`

## How to integrate with Travis CI

Configuring Travis CI involves constructing a `.travis.yml` file in the project root. This file, will contain all of the configuration
options for Travis CI in terms of what commands to run.

Refer to [Travis CI Getting Started](https://docs.travis-ci.com/user/getting-started) for more details on integrating with Travis CI.

```yml
dist: trusty

sudo: required

language: node_js
node_js:
  - "6"

addons:
  apt:
    packages:
    - google-chrome-stable

before_install:
  - export CHROME_BIN=/usr/bin/google-chrome

install:
  - npm install

before_script:
  - export DISPLAY=:99.0
  - sh -e /etc/init.d/xvfb start

script:
  - npm test
  - npm run e2e
```
