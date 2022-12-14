![cypress](https://img.shields.io/badge/Cypress%20version-10.3.1-brightgreen)

# Cypress @ PlayPlay

Play Play end-to-end automated testing tool.

## What is end-to-end testing about?

This is **not** about testing a complete user workflow, this is about testing a single feature going through every layer (front, back, DB,
external services) and check they all work well together.

## Table of contents

1. [Dependencies](#dependencies)
2. [Installation](#installation)
3. [Docker on CI](#docker-on-ci)
4. [Running the tests](#running-the-tests)
5. [Reports](#reports)
6. [Project](#project)
7. [Contributing](#contributing)
8. [Best practices](#best-practices)
9. [Data seeding](#data-seeding)
10. [Cucumber extension](#cucumber-extension)
11. [Troubleshooting](#troubleshooting)
12. [Known limitations](#known-limitations)
13. [Additional documentation](#additional-documentation)

## Dependencies

- Cypress 10.3.1
- Node 16.13.2 (shipped with Cypress)
- npm 8.16.0
- Python 3.9 and ffmpeg (for rendering tests)

## Installation

Note: If you are using MacOS with a M1 chip, you might have to install Rosetta to be able to use some libraries.
To do so, in your terminal type `softwareupdate --install-rosetta`
[Source Apple StackExchange](https://apple.stackexchange.com/questions/408375/zsh-bad-cpu-type-in-executable)
Then to install use `arch -x86_64 npm install` command or change Terminal mode with `arch -x86_64 bash` command

From the project root run `npm ci`
which will install all the dependencies without modifying the package-lock.json
`npm install` should only be used in case of installing new dependencies like a new version of cypress or adding a new library.

For rendering video comparison:

- Install python3, for example with Homebrew `brew install python@3.9` then restart your computer
- Linux systems only: install libsndfile `sudo apt-get install libsndfile1`
- Install python dependencies `pip3 install -r ./lib/compare_rendering/requirements.txt`
- Install ffmpeg, for example with Homebrew `brew install ffmpeg`

## Docker on CI

CI is using Docker to set up node and the browsers. If you want to upgrade one of these dependencies, just edit the `container:` line
in `ci-workflow.yml` with one of the images available on [Docker Hub](https://hub.docker.com/r/cypress/browsers/tags)

## Running the tests

### Basic commands

All the commands below will execute the tests from your machine and target the selected environment. Replace TARGET_ENV with your staging or
dev environment.

To open the Cypress graphical interface and choose which test to run, use `cypress open`:

```bash
npx cypress open -e baseUrl=https://TARGET_ENV -b chrome --e2e
```

To record your run on Cypress dashboard, use `--record --key`:

```bash
npx cypress run -e baseUrl=https://TARGET_ENV --record --key eb8ab670-eee4-4357-81c3-7d7551acc861
```

Refer to [Cypress command line documentation](https://docs.cypress.io/guides/guides/command-line#cypress-run) for more information on
command line arguments.

### Running a subset of tests using tags

To run a subset of test, you can use cucumber tags and add them to the run command:

```bash
npx cypress run -e baseUrl=https://TARGET_ENV,TAGS='not @firefoxOnly and @login'
```

You can combine tags with `and`, `or` `not`. Refer to [documention on cucumber tags](https://cucumber.io/docs/cucumber/api/#tags).

**Warning**

- To avoid running scenarios not adapted for the browser you want, don't forget to put the corresponding tag:
  - to run test on Chrome, ignore tests for Firefox with the tag `not @firefoxOnly`.
  - to run test on Firefox, ignore tests for Chrome with the tag `not @chromeOnly`.
- To disable some tests we tag them with @ignore. When launching the tests don't forget to use the tag `not @ignore`.

### Running rendering comparison tests

Rendering comparison tests are not included in the default config `cypress.config.ts` because we do not want to run them on the main suite.
A dedicated cypress config file must be used:

```bash
npx cypress run -e baseUrl=https://TARGET_ENV -C cypress_rendering.config.ts
```

### Running tests on GitHub Actions

[Check out the video tutorial](https://playplayfamily.slack.com/files/U01PSD521Q8/F02NRN35JTH/christmas-list.mp4)  
To run tests on the CI, go to the [GitHub Actions](https://github.com/playplay/functional-tests/actions/workflows/ci-workflow.yml) then
click on `Run workflow`.

Default build will run `all scenarios` on Chrome, on branch `main`. Make sure to replace `TARGET_ENV` with your staging or dev environment.

You can customize the build by selecting another branch, environment or cucumber tags.

## Reports

### Cypress dashboard

Find the credentials on [this notion page](https://www.notion.so/Quality-Assistance-4b36ab05726345a79858f8024fdf5cf1).  
Open the project on [Cypress dashboard](https://dashboard.cypress.io/projects/bnf64t/runs) then select a run. From the "Test results" tab,
you can get information about each test. On the right panel there is a link to the screenshot taken at the end of the test.  
![Cypress dashboard run report](resources/readme_reports.png)  
For failed tests only, there is also a recording of the execution and the stacktrace. When a test fails once, there is one retry attempt to
make it pass. A test that has failed then passed in the same run is marked as `Flaky` which indicates there is a random bug in the app or
the test is unstable.

### Slack

Any failed test run is posted on Slack as long as the test has been run on a staging environment. For example a run on staging4 will be
reported to `#cypress-staging4` channel if it failed.  
**Important:** Please open a thread on slack if you are the author of the run to explain why the run failed.

## Project

### Folders

Here is the architecture of this project:

- `api-request` - utility methods to call PlayPlay API
- `fixtures` - test data
- `integration` - cucumber main folder which contains:
  - `features` - the tests scenarios written with Gherkin language
  - `step-definitions` - the Javascript implementation of each Gherkin step
- `page-objects` - each file represents a PlayPlay page or a UI component
- `plugins` - additional Cypress plugins like cucumber preprocessor
- `support` - utility files (timeout values, Cypress commands, ...)

### Cypress config

Tweak Cypress and browser configuration in `cypress.config.ts` file.

### Eslint/Prettier Configuration

#### Visual Studio Code

- Add ESLint (Microsoft) and Prettier (Prettier) extensions by the Extensions manager of VS Code
- Configure "format on save": Settings -> Editor: check ** Format On Save **
- Configure (add to) your settings.json (to find it, see https://code.visualstudio.com/docs/getstarted/settings#_settings-file-locations)

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  "editor.formatOnPaste": false
}
```

#### Intellij IDEA Ultimate

- Install Prettier (JetBrains) plugin. ESLint should already be installed (natively, it's unlisted)
- in Settings, Configure "format on save" (Tools > Actions on Save) and:
  - Reformat code
  - Run eslint --fix
  - Run Prettier
  - click on configure ESLint then enable "Automatic ESLint configuration"
  - go back, click on configure Prettier then enable On 'Reformat Code' action and On save
- Go to Editor > Code Style > Javascript > set tab size at 2

## Contributing

### Feature branch

Create a new branch from `develop`, push, make a pull request then run the tests from GitHub Actions against this branch.  
The PR needs at least 1 approval and successful tests results without flakiness to be merged. Once it is merged to `develop` it will later
be merged to `main` when declared fully stable.

### Hotfix branch

If a test is broken and can easily be fixed (locators to update, change a simple data set, etc.), this should be done asap to prevent a fail
on the developers PR validation:

- create a hotfix branch `hotfix/name-of-fix`
- commit, push, create a pull request then run the tests from GitHub Actions
- **code review not required in this case**, merge your code asap

## Best practices

### Layers responsibility

features (gherkin) -> step-definitions (js) -> page-objects (js)

- [Gherkin](#writing-your-first-scenario-with-gherkin) : this is the `.feature` file. Here you describe the use case as if you were a user
  without technical knowledge. Everyone in the company should be able to understand and reproduce the test case. Here, you avoid talking
  about the way the feature is implemented, for example "the user logs in"
  instead of "the user fills ..."
- [Step definition](#step-definition) : here you stick to the interface and describe each interaction. Anyone that reads the code should be
  able to reproduce the test case as if it was a script. This is also where you write assertions.
- [Page object](#page-object) : each interaction is a page object (see below). You should avoid adding logic and assertions here, keep it
  simple and stupid ;)

### Writing your first scenario with Gherkin

Gherkin is a non-technical and human-readable language used to write a test scenario. It is interpreted by Cucumber to make the link with
the Javascript test definition. Check out what you can do with Gherkin in this [documentation](https://cucumber.io/docs/gherkin/reference/),
you should definitely know how to use these keywords before writing you first scenario :

- Feature
- Background
- Scenario
- Scenario Outline and Examples
- Step Arguments (doc string, data tables, etc.) and Cucumber Expressions in
  this [dedicated documentation](https://github.com/cucumber/cucumber-expressions#readme)

At PlayPlay we try to restrict to **1 scenario per feature file** in order to keep a fast CI execution (only feature files are parallelized)

Add a `@tagName` at the top of you feature file to be able to easily run a subset of the test suite later on.

Use `Given`, `When`, `Then`, `And`, `But` in correct order to make your scenario easy to read:

- `Given` precondition
- `When` action
- `Then` expected result and assertion
- `And` `But` to link steps

**Reuse** steps: when typing a word, your IDE should give you hints about existing steps (may require installing Gherkin or Cucumber plugin)
. You can also use step parameters to call the same steps but with different data.

For new steps, please follow these rules:

- start with lowercase
- describe who does what: `the user fills in his credentials`
- describe what should be asserted: `the media duration should be "00:07:86"`

You can then declare the definition of your new step in JS files.

### Step definition

These are organized by feature, page and UI component and should end with `...Steps.js`: authenticationStep.js, templatesSteps.js,
navBarSteps.js  
A step definition can start with any Gherkin keyword (Given, When, Then => doesn't matter) followed by the cucumber expression. It should be
short, call Page Object methods then do an assertion.

### Page object

Page object is a common testing pattern that describes the features of a page in the app into a single file. For example the login page
is ` page-objects/signUpPage.js`  
Place the [locators](#locators) at the top of the file. Try to create one method per action:

- click a button
- fill a form
- wait for the page to be loaded

It is a good practice to then break things into UI components when possible, so page components can be then imported to other page objects.
For example the top navigation bar is almost always visible in the app, it could have its own `js` file then be imported.

#### Locators

These constants allow to identify an element in the DOM. Pick the correct locator using these priorities:

1. [data attributes](https://www.w3schools.com/tags/att_data-.asp) => only `data-heap-label` should be used
2. [other css selector](https://www.w3schools.com/cssref/css_selectors.asp) like `#id` or `.class`

Xpath use is strictly forbidden, if there is no `data-heap-label` or unique css selector, it could be the time to add a `data-heap-label`
directly into PlayPlay's code! Ask a developer or another QA member to help you. A `.class` locator may identify multiple elements on the
page, try to restrict the search to a section of the page:

```javascript
cy.get(locators.pageSection).find(locators.someButton);
```

**Stable locators reduce maintenance effort a lot.**

### How to wait

Bad wait code is the main source of flakiness in e2e tests, be careful! Bad practice consists in:

- [conditional testing on unstable DOM](https://docs.cypress.io/guides/core-concepts/conditional-testing)
- waiting for an element to disappear using "this element should not be visible" when the element has not be shown yet => the test passes
  too early
- using `cy.wait(1000)` for an element to be visible => you don't really know when the element will show up so this will fail when the app
  is a little bit too slow

Instead:

1. take advantage of Cypress intercept to wait for the server response before asserting UI elements
2. do not wait for unstable DOM elements like animations, try to think of other ways to wait for the app to reach a certain state
3. also define timeout constants with good naming in `support/timeouts.js`, use short timeout so the tests does not hang up for too long if
   a bug prevents the element to show up

#### Intercept, endpoints and aliases

In PlayPlay we favor intercepts to handle variable wait due to media processing. We have utility methods in `intercepts` folder to call and
wait using intercept:

- the `registerXXX` methods are able to register an endpoint only once and prevent Cypress issues with duplicated intercepts
- the `waitXXX` methods are able to wait for a registered intercept to be matched

To learn more about Cypress aliases and how to use them outside of intercept, please read
the [Variables and Aliases doc](https://docs.cypress.io/guides/core-concepts/variables-and-aliases).

### Cucumber extension

This extension adds rich language support for the Cucumber (Gherkin) language in order to facilitate tests implementation. It includes:

- Syntax highlight
- Auto-parsing of feature steps from paths, provided in .vscode/settings.json
- Autocompletion of steps
- Ontype validation for all the steps
- Definitions support for all the steps parts

#### Visual Studio Code

![Vscode Cucumber extension](resources/vscode_cucumber_extension.png)
To use it in VS Code:

- Open VS Code Extensions Marketplace (Ctrl+Shift+X) and Install the extension
- In the opened app root create `.vscode` folder with settings.json file
- Add all the following settings to the settings.json file

```json
{
  "cucumberautocomplete.steps": [
    "cypress/integration/step-definitions/**/*.js"
  ],
  "cucumberautocomplete.strictGherkinCompletion": true
}
```

- Reload app to apply all the extension changes

#### Intellij IDEA

In IntelliJ Ultimate, the required plugins are bundled and enabled by default.  
In IntelliJ Community, the necessary plugins are not bundled, that is why you need to install and enable them:

- In the Settings/Preferences dialog, select Plugins
- Switch to the Marketplace tab and install [Gherkin](https://plugins.jetbrains.com/plugin/9164-gherkin)
  and [Cucumber.js](https://plugins.jetbrains.com/plugin/7418-cucumber-js) if they are not present in the Install tab
- Apply the changes and close the dialog. Restart the IDE if prompted.

## Data seeding

Data seeding is the act of creating an initial set of “dummy” data on a database.

### Quick tour

Before running the Cypress tests in CI, there’s a call to the API endpoint GET /api/seed-data, which begins the full process of seeding the
database. When the endpoint is called, the server generates a timestamp that is then used to identify the data created by seeding.
This timestamp is used to indentify all the users created by the data seeding, all the refernces are in cypress/fixtures/users.json.

### How to use it

The different accounts are provided by functions in cypress/integration/utils-for-steps/providerDsUser.js.

#### In CI

In CI the timestamp is given to the Cypress project (from the github workflow) by an environement variable `Cypress.env('timestamp')`.

#### In Local

In local the temporary soltion is to fill the Cypress environement variable `Cypress.env('timestamp')` in command
line `npm run cypress-ds -- -e baseUrl=https://TARGET_ENV,timestamp=TARGET_ENV_DS_TIMESTAMP` by finding the timestamp in the account already
created on the target environement TARGET_ENV.

## Add a new media to our test suite

### Add the media to the project

To be able to use a new media, you need to put it in the project. Two steps for doing that:

1. Upload your media into the functional-tests/cypress/fixtures/medias directory
2. Add the media infos to functional-tests/cypress/fixtures/medias.json file

To edit medias.json, you'll need to know the size and hash of your file.
**To know the size :** simply do a ls -al <filename> and copy the given size
**To know the hash :** use the command md5sum <filename> or md5 <filename> and copy the given hash

### Ask the devops to upload the file in the storage

In order to be able to use this new file correctly, it must be added to the proper storage. To do so, you'll have to ask the devops to
upload the file on https://storage.googleapis.com/playplay-onthefly/tests-e2e-assets.
Post a message on the #devops slack channel, asking them to upload your file on the repo. Don't forget to give them the file ;)

### Upload your file on production env once

You have to do it for your file to be in the Copy Prod Data process.

### Update the Postman collection

Finally, you'll have to update the Postman Collection and query the raw-medias endpoint. To do so, follow those steps:

1. If you don't have it, import the dedicated Postman collection and env configuration you'll find in `playplay` repository
   under `ci/seeding`
2. Duplicate an existing request in Setup medias/Create raw medias collection
3. Update its body with your file informations
4. Update the second test with an id for your file
5. Put a correct baseUrl with an env where your file have been uploaded
6. Send the request

That's it, you're done! You can now launch your feature test in any environment where there have been a Copy Prod Data after you uploaded
your file on Production. If the environement you want to use did not have a CPD, just upload the file on this env and your test will work.

## Troubleshooting

Check out [Cypress troubleshooting documentation](https://docs.cypress.io/guides/references/troubleshooting).

### cypress failed to connect to chrome

If chrome is opened during the test but cypress cannot communicate with it, clear cypress cache: `cypress cache clear`

## Known limitations

- File download is currently not working for Firefox. Test scenarios have been adapted to remove the download steps. More investigation is
  needed to make it work.
- Due to Cypress' [same-origin](https://docs.cypress.io/guides/references/trade-offs.html#Same-origin) policy, scenarios using iframe on the
  payment page are not working with Firefox and therefore have been removed from the test suite.
- Cypress proxy is slow at displaying images, especially since version 8. There is
  an [open issue](https://github.com/cypress-io/cypress/issues/18771) about this that may be addressed but meanwhile we have increased the
  timeouts in `cypress.config.ts` to prevent flakiness.
- Unsplash stock API is [limited to 50 calls per hour](https://unsplash.com/documentation#registering-your-application), if this limit is
  reached you have to wait for it to be reset or ignore the tests with `@unsplash` tag.

## Additional documentation

- [Cypress best practices](https://docs.cypress.io/guides/references/best-practices)
- [Debugging Cypress](https://docs.cypress.io/guides/guides/debugging)
- [Useful automation patterns](https://testautomationpatterns.org/wiki/index.php/Test_Automation_Patterns_Mind_Map)

## GitHub Actions

The GitHub Actions files are located under the **.github/** folder.

The main workflow can be started from the UI.
![Start tests](resources/run_action.png)

You need to provide the environment url you want to test and modify the test tags if is needed.
The input "ID of the PR on PlayPlay repository" is not mandatory because he is used by the PlayPlay CI to run tests for an env on-the-fly.

Global presentation of the workflow jobs :

- **prepare-with-pr** : this job (executed if we are on the pull request mode) is used to retrieve informations about the pull request (
  author, commit message ...) and add a status on the last commit of the pull request (on the playplay repository)
- **prepare** : this job prepare things (autoscale environment, generate uniq ID) before running the cypress tests
- **test** : this job setup context and run the tests (this job is parallelized)
- **finalize** : this job (always executed but skipped if we are not on the pull request mode) add a status on the last commit of the pull
  request with the status of this CI (success or failure)

The workflow has two "mode" :

- the first mode "normal" when the PR ID is not provided
- the second mode "pull request" when the PR ID is provided

### Normal mode (PR-ID not provided)

When the workflow is started without the pull request ID, we start the tests in the context of the "functional-tests" repository.
In the Cypress Dashboard, every test is associated with the commit on the "functional-tests" repository.
Tests are sended on the Cypress dashboard under the "End-to-end" project.

### Pull request mode (PR-ID provided)

When the workflow is started with a pull request ID, we start the tests in the context of the "playplay" repository.
In the Cypress Dashboard, every test is associated with the commit on the "playplay" repository.
The test are sended on the Cypress dashboard under the "playplay" project.
