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

🔗 [**Read More: Increase transparency using smart logging**](/sections/production/smartlogging.md)

<br/><br/>

## 5.3. Delegate anything possible (e.g. gzip, SSL) to a reverse proxy

**TL;DR:** Node is awfully bad at doing CPU intensive tasks like gzipping, SSL termination, etc. You should use ‘real’ middleware services like nginx, HAproxy or cloud vendor services instead

**Otherwise:** Your poor single thread will stay busy doing infrastructural tasks instead of dealing with your application core and performance will degrade accordingly

🔗 [**Read More: Delegate anything possible (e.g. gzip, SSL) to a reverse proxy**](/sections/production/delegatetoproxy.md)

<br/><br/>

## 5.4. Lock dependencies

**TL;DR:** Your code must be identical across all environments, but amazingly npm lets dependencies drift across environments by default – when you install packages at various environments it tries to fetch packages’ latest patch version. Overcome this by using npm config files, .npmrc, that tell each environment to save the exact (not the latest) version of each package. Alternatively, for finer grained control use `npm shrinkwrap`. \*Update: as of NPM5, dependencies are locked by default. The new package manager in town, Yarn, also got us covered by default

**Otherwise:** QA will thoroughly test the code and approve a version that will behave differently in production. Even worse, different servers in the same production cluster might run different code

🔗 [**Read More: Lock dependencies**](/sections/production/lockdependencies.md)

<br/><br/>

## 5.5. Guard process uptime using the right tool

**TL;DR:** The process must go on and get restarted upon failures. For simple scenarios, process management tools like PM2 might be enough but in today's ‘dockerized’ world, cluster management tools should be considered as well

**Otherwise:** Running dozens of instances without a clear strategy and too many tools together (cluster management, docker, PM2) might lead to DevOps chaos

🔗 [**Read More: Guard process uptime using the right tool**](/sections/production/guardprocess.md)

<br/><br/>

## 5.6. Utilize all CPU cores

**TL;DR:** At its basic form, a Node app runs on a single CPU core while all others are left idling. It’s your duty to replicate the Node process and utilize all CPUs – For small-medium apps you may use Node Cluster or PM2. For a larger app consider replicating the process using some Docker cluster (e.g. K8S, ECS) or deployment scripts that are based on Linux init system (e.g. systemd)

**Otherwise:** Your app will likely utilize only 25% of its available resources(!) or even less. Note that a typical server has 4 CPU cores or more, naive deployment of Node.js utilizes only 1 (even using PaaS services like AWS beanstalk!)

🔗 [**Read More: Utilize all CPU cores**](/sections/production/utilizecpu.md)

<br/><br/>

## 5.7. Create a ‘maintenance endpoint’

**TL;DR:** Expose a set of system-related information, like memory usage and REPL, etc in a secured API. Although it’s highly recommended to rely on standard and battle-tests tools, some valuable information and operations are easier done using code

**Otherwise:** You’ll find that you’re performing many “diagnostic deploys” – shipping code to production only to extract some information for diagnostic purposes

🔗 [**Read More: Create a ‘maintenance endpoint’**](/sections/production/createmaintenanceendpoint.md)

<br/><br/>

## 5.8. Discover errors and downtime using APM products

**TL;DR:** Application monitoring and performance products (a.k.a APM) proactively gauge codebase and API so they can auto-magically go beyond traditional monitoring and measure the overall user-experience across services and tiers. For example, some APM products can highlight a transaction that loads too slow on the end-users side while suggesting the root cause

**Otherwise:** You might spend great effort on measuring API performance and downtimes, probably you’ll never be aware which is your slowest code parts under real-world scenario and how these affect the UX

🔗 [**Read More: Discover errors and downtime using APM products**](/sections/production/apmproducts.md)

<br/><br/>

## 5.9. Make your code production-ready

**TL;DR:** Code with the end in mind, plan for production from day 1. This sounds a bit vague so I’ve compiled a few development tips that are closely related to production maintenance (click Gist below)

**Otherwise:** A world champion IT/DevOps guy won’t save a system that is badly written

🔗 [**Read More: Make your code production-ready**](/sections/production/productioncode.md)

<br/><br/>

## 5.10. Measure and guard the memory usage

**TL;DR:** Node.js has controversial relationships with memory: the v8 engine has soft limits on memory usage (1.4GB) and there are known paths to leak memory in Node’s code – thus watching Node’s process memory is a must. In small apps, you may gauge memory periodically using shell commands but in medium-large apps consider baking your memory watch into a robust monitoring system

**Otherwise:** Your process memory might leak a hundred megabytes a day like how it happened at [Walmart](https://www.joyent.com/blog/walmart-node-js-memory-leak)

🔗 [**Read More: Measure and guard the memory usage**](/sections/production/measurememory.md)

<br/><br/>

## 5.11. Get your frontend assets out of Node

**TL;DR:** Serve frontend content using dedicated middleware (nginx, S3, CDN) because Node performance really gets hurt when dealing with many static files due to its single-threaded model

**Otherwise:** Your single Node thread will be busy streaming hundreds of html/images/angular/react files instead of allocating all its resources for the task it was born for – serving dynamic content

🔗 [**Read More: Get your frontend assets out of Node**](/sections/production/frontendout.md)

<br/><br/>

## 5.12. Be stateless, kill your servers almost every day

**TL;DR:** Store any type of data (e.g. user sessions, cache, uploaded files) within external data stores. Consider ‘killing’ your servers periodically or use ‘serverless’ platform (e.g. AWS Lambda) that explicitly enforces a stateless behavior

**Otherwise:** Failure at a given server will result in application downtime instead of just killing a faulty machine. Moreover, scaling-out elasticity will get more challenging due to the reliance on a specific server

🔗 [**Read More: Be stateless, kill your Servers almost every day**](/sections/production/bestateless.md)

<br/><br/>

## 5.13. Use tools that automatically detect vulnerabilities

**TL;DR:** Even the most reputable dependencies such as Express have known vulnerabilities (from time to time) that can put a system at risk. This can be easily tamed using community and commercial tools that constantly check for vulnerabilities and warn (locally or at GitHub), some can even patch them immediately

**Otherwise:** Keeping your code clean from vulnerabilities without dedicated tools will require you to constantly follow online publications about new threats. Quite tedious

🔗 [**Read More: Use tools that automatically detect vulnerabilities**](/sections/production/detectvulnerabilities.md)

<br/><br/>

## 5.14. Assign a transaction id to each log statement

**TL;DR:** Assign the same identifier, transaction-id: {some value}, to each log entry within a single request. Then when inspecting errors in logs, easily conclude what happened before and after. Unfortunately, this is not easy to achieve in Node due to its async nature, see code examples inside

**Otherwise:** Looking at a production error log without the context – what happened before – makes it much harder and slower to reason about the issue

🔗 [**Read More: Assign ‘TransactionId’ to each log statement**](/sections/production/assigntransactionid.md)

<br/><br/>

## 5.15. Set NODE_ENV=production

**TL;DR:** Set the environment variable NODE_ENV to ‘production’ or ‘development’ to flag whether production optimizations should get activated – many npm packages determine the current environment and optimize their code for production

**Otherwise:** Omitting this simple property might greatly degrade performance. For example, when using Express for server-side rendering omitting `NODE_ENV` makes it slower by a factor of three!

🔗 [**Read More: Set NODE_ENV=production**](/sections/production/setnodeenv.md)

<br/><br/>

## 5.16. Design automated, atomic and zero-downtime deployments

**TL;DR:** Research shows that teams who perform many deployments lower the probability of severe production issues. Fast and automated deployments that don’t require risky manual steps and service downtime significantly improve the deployment process. You should probably achieve this using Docker combined with CI tools as they became the industry standard for streamlined deployment

**Otherwise:** Long deployments -> production downtime & human-related error -> team unconfident in making deployment -> fewer deployments and features

<br/><br/>

## 5.17. Use an LTS release of Node.js

**TL;DR:** Ensure you are using an LTS version of Node.js to receive critical bug fixes, security updates and performance improvements

**Otherwise:** Newly discovered bugs or vulnerabilities could be used to exploit an application running in production, and your application may become unsupported by various modules and harder to maintain

🔗 [**Read More: Use an LTS release of Node.js**](/sections/production/LTSrelease.md)

<br/><br/>

## 5.18. Don't route logs within the app

**TL;DR:** Log destinations should not be hard-coded by developers within the application code, but instead should be defined by the execution environment the application runs in. Developers should write logs to `stdout` using a logger utility and then let the execution environment (container, server, etc.) pipe the `stdout` stream to the appropriate destination (i.e. Splunk, Graylog, ElasticSearch, etc.).

**Otherwise:** Application handling log routing === hard to scale, loss of logs, poor separation of concerns

🔗 [**Read More: Log Routing**](/sections/production/logrouting.md)

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

🔗 [**Read More: Lint rules**](/sections/security/lintrules.md)

<br/><br/>

## 6.2. Limit concurrent requests using a middleware

<a href="https://www.owasp.org/index.php/Denial_of_Service" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** DOS attacks are very popular and relatively easy to conduct. Implement rate limiting using an external service such as cloud load balancers, cloud firewalls, nginx, [rate-limiter-flexible](https://www.npmjs.com/package/rate-limiter-flexible) package, or (for smaller and less critical apps) a rate-limiting middleware (e.g. [express-rate-limit](https://www.npmjs.com/package/express-rate-limit))

**Otherwise:** An application could be subject to an attack resulting in a denial of service where real users receive a degraded or unavailable service.

🔗 [**Read More: Implement rate limiting**](/sections/security/limitrequests.md)

<br/><br/>

## 6.3 Extract secrets from config files or use packages to encrypt them

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A3-Sensitive_Data_Exposure" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A3:Sensitive%20Data%20Exposure%20-green.svg" alt=""/></a>

**TL;DR:** Never store plain-text secrets in configuration files or source code. Instead, make use of secret-management systems like Vault products, Kubernetes/Docker Secrets, or using environment variables. As a last resort, secrets stored in source control must be encrypted and managed (rolling keys, expiring, auditing, etc). Make use of pre-commit/push hooks to prevent committing secrets accidentally

**Otherwise:** Source control, even for private repositories, can mistakenly be made public, at which point all secrets are exposed. Access to source control for an external party will inadvertently provide access to related systems (databases, apis, services, etc).

🔗 [**Read More: Secret management**](/sections/security/secretmanagement.md)

<br/><br/>

## 6.4. Prevent query injection vulnerabilities with ORM/ODM libraries

<a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a>

**TL;DR:** To prevent SQL/NoSQL injection and other malicious attacks, always make use of an ORM/ODM or a database library that escapes data or supports named or indexed parameterized queries, and takes care of validating user input for expected types. Never just use JavaScript template strings or string concatenation to inject values into queries as this opens your application to a wide spectrum of vulnerabilities. All the reputable Node.js data access libraries (e.g. [Sequelize](https://github.com/sequelize/sequelize), [Knex](https://github.com/tgriesser/knex), [mongoose](https://github.com/Automattic/mongoose)) have built-in protection against injection attacks.

**Otherwise:** Unvalidated or unsanitized user input could lead to operator injection when working with MongoDB for NoSQL, and not using a proper sanitization system or ORM will easily allow SQL injection attacks, creating a giant vulnerability.

🔗 [**Read More: Query injection prevention using ORM/ODM libraries**](/sections/security/ormodmusage.md)

<br/><br/>

## 6.5. Collection of generic security best practices

**TL;DR:** This is a collection of security advice that is not related directly to Node.js - the Node implementation is not much different than any other language. Click read more to skim through.

🔗 [**Read More: Common security best practices**](/sections/security/commonsecuritybestpractices.md)

<br/><br/>

## 6.6. Adjust the HTTP response headers for enhanced security

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** Your application should be using secure headers to prevent attackers from using common attacks like cross-site scripting (XSS), clickjacking and other malicious attacks. These can be configured easily using modules like [helmet](https://www.npmjs.com/package/helmet).

**Otherwise:** Attackers could perform direct attacks on your application's users, leading to huge security vulnerabilities

🔗 [**Read More: Using secure headers in your application**](/sections/security/secureheaders.md)

<br/><br/>

## 6.7. Constantly and automatically inspect for vulnerable dependencies

<a href="https://www.owasp.org/index.php/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Known%20Vulnerabilities%20-green.svg" alt=""/></a>

**TL;DR:** With the npm ecosystem it is common to have many dependencies for a project. Dependencies should always be kept in check as new vulnerabilities are found. Use tools like [npm audit](https://docs.npmjs.com/cli/audit) or [snyk](https://snyk.io/) to track, monitor and patch vulnerable dependencies. Integrate these tools with your CI setup so you catch a vulnerable dependency before it makes it to production.

**Otherwise:** An attacker could detect your web framework and attack all its known vulnerabilities.

🔗 [**Read More: Dependency security**](/sections/security/dependencysecurity.md)

<br/><br/>

## 6.8. Avoid using the Node.js crypto library for handling passwords, use Bcrypt

<a href="https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Broken%20Authentication%20-green.svg" alt=""/></a>

**TL;DR:** Passwords or secrets (API keys) should be stored using a secure hash + salt function like `bcrypt`, that should be a preferred choice over its JavaScript implementation due to performance and security reasons.

**Otherwise:** Passwords or secrets that are persisted without using a secure function are vulnerable to brute forcing and dictionary attacks that will lead to their disclosure eventually.

🔗 [**Read More: Use Bcrypt**](/sections/security/bcryptpasswords.md)

<br/><br/>

## 6.9. Escape HTML, JS and CSS output

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a>

**TL;DR:** Untrusted data that is sent down to the browser might get executed instead of just being displayed, this is commonly referred as a cross-site-scripting (XSS) attack. Mitigate this by using dedicated libraries that explicitly mark the data as pure content that should never get executed (i.e. encoding, escaping)

**Otherwise:** An attacker might store malicious JavaScript code in your DB which will then be sent as-is to the poor clients

🔗 [**Read More: Escape output**](/sections/security/escape-output.md)

<br/><br/>

## 6.10. Validate incoming JSON schemas

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7: XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A8-Insecure_Deserialization" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A8:Insecured%20Deserialization%20-green.svg" alt=""/></a>

**TL;DR:** Validate the incoming requests' body payload and ensure it meets expectations, fail fast if it doesn't. To avoid tedious validation coding within each route you may use lightweight JSON-based validation schemas such as [jsonschema](https://www.npmjs.com/package/jsonschema) or [joi](https://www.npmjs.com/package/joi)

**Otherwise:** Your generosity and permissive approach greatly increases the attack surface and encourages the attacker to try out many inputs until they find some combination to crash the application

🔗 [**Read More: Validate incoming JSON schemas**](/sections/security/validation.md)

<br/><br/>

## 6.11. Support blacklisting JWTs

<a href="https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Broken%20Authentication%20-green.svg" alt=""/></a>

**TL;DR:** When using JSON Web Tokens (for example, with [Passport.js](https://github.com/jaredhanson/passport)), by default there's no mechanism to revoke access from issued tokens. Once you discover some malicious user activity, there's no way to stop them from accessing the system as long as they hold a valid token. Mitigate this by implementing a blacklist of untrusted tokens that are validated on each request.

**Otherwise:** Expired, or misplaced tokens could be used maliciously by a third party to access an application and impersonate the owner of the token.

🔗 [**Read More: Blacklist JSON Web Tokens**](/sections/security/expirejwt.md)

<br/><br/>

## 6.12. Prevent brute-force attacks against authorization

<a href="https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A9:Broken%20Authentication%20-green.svg" alt=""/></a>

**TL;DR:** A simple and powerful technique is to limit authorization attempts using two metrics:
           
1. The first is number of consecutive failed attempts by the same user unique ID/name and IP address.
2. The second is number of failed attempts from an IP address over some long period of time. For example, block an IP address if it makes 100 failed attempts in one day.

**Otherwise:** An attacker can issue unlimited automated password attempts to gain access to privileged accounts on an application

🔗 [**Read More: Login rate limiting**](/sections/security/login-rate-limit.md)

<br/><br/>

## 6.13. Run Node.js as non-root user

<a href="https://www.owasp.org/index.php/Top_10-2017_A5-Broken_Access_Control" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A5:Broken%20Access%20Access%20Control-green.svg" alt=""/></a>

**TL;DR:** There is a common scenario where Node.js runs as a root user with unlimited permissions. For example, this is the default behaviour in Docker containers. It's recommended to create a non-root user and either bake it into the Docker image (examples given below) or run the process on this user's behalf by invoking the container with the flag "-u username"

**Otherwise:** An attacker who manages to run a script on the server gets unlimited power over the local machine (e.g. change iptable and re-route traffic to his server)

🔗 [**Read More: Run Node.js as non-root user**](/sections/security/non-root-user.md)

<br/><br/>

## 6.14. Limit payload size using a reverse-proxy or a middleware

<a href="https://www.owasp.org/index.php/Top_10-2017_A8-Insecure_Deserialization" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A8:Insecured%20Deserialization%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** The bigger the body payload is, the harder your single thread works in processing it. This is an opportunity for attackers to bring servers to their knees without tremendous amount of requests (DOS/DDOS attacks). Mitigate this limiting the body size of incoming requests on the edge (e.g. firewall, ELB) or by configuring [express body parser](https://github.com/expressjs/body-parser) to accept only small-size payloads

**Otherwise:** Your application will have to deal with large requests, unable to process the other important work it has to accomplish, leading to performance implications and vulnerability towards DOS attacks

🔗 [**Read More: Limit payload size**](/sections/security/requestpayloadsizelimit.md)

<br/><br/>

## 6.15. Avoid JavaScript eval statements

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** `eval` is evil as it allows executing custom JavaScript code during run time. This is not just a performance concern but also an important security concern due to malicious JavaScript code that may be sourced from user input. Another language feature that should be avoided is `new Function` constructor. `setTimeout` and `setInterval` should never be passed dynamic JavaScript code either.

**Otherwise:** Malicious JavaScript code finds a way into text passed into `eval` or other real-time evaluating JavaScript language functions, and will gain complete access to JavaScript permissions on the page. This vulnerability is often manifested as an XSS attack.

🔗 [**Read More: Avoid JavaScript eval statements**](/sections/security/avoideval.md)

<br/><br/>

## 6.16. Prevent evil RegEx from overloading your single thread execution

<a href="https://www.owasp.org/index.php/Denial_of_Service" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20DDOS%20-green.svg" alt=""/></a>

**TL;DR:** Regular Expressions, while being handy, pose a real threat to JavaScript applications at large, and the Node.js platform in particular. A user input for text to match might require an outstanding amount of CPU cycles to process. RegEx processing might be inefficient to an extent that a single request that validates 10 words can block the entire event loop for 6 seconds and set the CPU on 🔥. For that reason, prefer third-party validation packages like [validator.js](https://github.com/chriso/validator.js) instead of writing your own Regex patterns, or make use of [safe-regex](https://github.com/substack/safe-regex) to detect vulnerable regex patterns

**Otherwise:** Poorly written regexes could be susceptible to Regular Expression DoS attacks that will block the event loop completely. For example, the popular `moment` package was found vulnerable with malicious RegEx usage in November of 2017

🔗 [**Read More: Prevent malicious RegEx**](/sections/security/regex.md)

<br/><br/>

## 6.17. Avoid module loading using a variable

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** Avoid requiring/importing another file with a path that was given as parameter due to the concern that it could have originated from user input. This rule can be extended for accessing files in general (i.e. `fs.readFile()`) or other sensitive resource access with dynamic variables originating from user input. [Eslint-plugin-security](https://www.npmjs.com/package/eslint-plugin-security) linter can catch such patterns and warn early enough

**Otherwise:** Malicious user input could find its way to a parameter that is used to require tampered files, for example, a previously uploaded file on the filesystem, or access already existing system files.

🔗 [**Read More: Safe module loading**](/sections/security/safemoduleloading.md)

<br/><br/>

## 6.18. Run unsafe code in a sandbox

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** When tasked to run external code that is given at run-time (e.g. plugin), use any sort of 'sandbox' execution environment that isolates and guards the main code against the plugin. This can be achieved using a dedicated process (e.g. `cluster.fork()`), serverless environment or dedicated npm packages that act as a sandbox

**Otherwise:** A plugin can attack through an endless variety of options like infinite loops, memory overloading, and access to sensitive process environment variables

🔗 [**Read More: Run unsafe code in a sandbox**](/sections/security/sandbox.md)

<br/><br/>

## 6.19. Take extra care when working with child processes

<a href="https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A7:XSS%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A1-Injection" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A1:Injection%20-green.svg" alt=""/></a> <a href="https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A4:External%20Entities%20-green.svg" alt=""/></a>

**TL;DR:** Avoid using child processes when possible and validate and sanitize input to mitigate shell injection attacks if you still have to. Prefer using `child_process.execFile` which by definition will only execute a single command with a set of attributes and will not allow shell parameter expansion.

**Otherwise:** Naive use of child processes could result in remote command execution or shell injection attacks due to malicious user input passed to an unsanitized system command.

🔗 [**Read More: Be cautious when working with child processes**](/sections/security/childprocesses.md)

<br/><br/>

## 6.20. Hide error details from clients

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** An integrated express error handler hides the error details by default. However, great are the chances that you implement your own error handling logic with custom Error objects (considered by many as a best practice). If you do so, ensure not to return the entire Error object to the client, which might contain some sensitive application details

**Otherwise:** Sensitive application details such as server file paths, third party modules in use, and other internal workflows of the application which could be exploited by an attacker, could be leaked from information found in a stack trace

🔗 [**Read More: Hide error details from client**](/sections/security/hideerrors.md)

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

🔗 [**Read More: Cookie and session security**](/sections/security/sessions.md)

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

🔗 [**Read More: Prevent unsafe redirects**](/sections/security/saferedirects.md)

<br/><br/>

## 6.25. Avoid publishing secrets to the npm registry

<a href="https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration" target="_blank"><img src="https://img.shields.io/badge/%E2%9C%94%20OWASP%20Threats%20-%20A6:Security%20Misconfiguration%20-green.svg" alt=""/></a>

**TL;DR:** Precautions should be taken to avoid the risk of accidentally publishing secrets to public npm registries. An `.npmignore` file can be used to blacklist specific files or folders, or the `files` array in `package.json` can act as a whitelist.

**Otherwise:** Your project's API keys, passwords or other secrets are open to be abused by anyone who comes across them, which may result in financial loss, impersonation, and other risks.

🔗 [**Read More: Avoid publishing secrets**](/sections/security/avoid_publishing_secrets.md)
<br/><br/><br/>

<p align="right"><a href="#table-of-contents">⬆ Return to top</a></p>

# `7. Performance Best Practices`

## 7.1. Prefer native JS methods over user-land utils like Lodash

 **TL;DR:** It's often more penalising to use utility libraries like `lodash` and `underscore` over native methods as it leads to unneeded dependencies and slower performance.
 Bear in mind that with the introduction of the new V8 engine alongside the new ES standards, native methods were improved in such a way that it's now about 50% more performant than utility libraries.

**Otherwise:** You'll have to maintain less performant projects where you could have simply used what was **already** available or dealt with a few more lines in exchange of a few more files.

🔗 [**Read More: Native over user land utils**](/sections/performance/nativeoverutil.md)

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

### What Other Bloggers Say

From the blog [Rising Stack](http://mubaloo.com/best-practices-deploying-node-js-applications/):

> …We recommend you to watch these signals for all of your services:
> Error Rate: Because errors are user facing and immediately affect your customers.
> Response time: Because the latency directly affects your customers and business.
> Throughput: The traffic helps you to understand the context of increased error rates and the latency too.
> Saturation: It tells how “full” your service is. If the CPU usage is 90%, can your system handle more traffic? …

<br></br>
