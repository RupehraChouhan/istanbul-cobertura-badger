istanbul-cobertura-badger
=========================
[![Travis](https://api.travis-ci.org/intuit/istanbul-cobertura-badger.svg)](https://travis-ci.org/intuit/istanbul-cobertura-badger) [![npm version](https://badge.fury.io/js/istanbul-cobertura-badger.svg)](http://badge.fury.io/js/istanbul-cobertura-badger) [![Codacy Badge](https://www.codacy.com/project/badge/172621abbd81457d84ee5df6ebe13f91)](https://www.codacy.com/app/marcellodesales/istanbul-cobertura-badger) [![Code Climate](https://codeclimate.com/github/marcellodesales/istanbul-cobertura-badger/badges/gpa.svg)](https://codeclimate.com/github/marcellodesales/istanbul-cobertura-badger) [![Dependency Status](https://david-dm.org/intuit/istanbul-cobertura-badger.svg)](https://david-dm.org/intuit/istanbul-cobertura-badger) [![devDependency Status](https://david-dm.org/intuit/istanbul-cobertura-badger/dev-status.svg)](https://david-dm.org/intuit/istanbul-cobertura-badger#info=devDependencies) [![Coverage Status](https://coveralls.io/repos/intuit/istanbul-cobertura-badger/badge.svg?branch=master&service=github)](https://coveralls.io/github/intuit/istanbul-cobertura-badger?branch=master) ![License](https://img.shields.io/badge/license-MIT-lightgray.svg)

Creates a coverage badge by reading the Cobertura XML coverage report from node-istanbul reports using  https://github.com/badges/shields.

[![NPM](https://nodei.co/npm/istanbul-cobertura-badger.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/istanbul-cobertura-badger/)

Requirements
========

I started to work with Node a few months ago, and I recently stumbled upon many different badges on GitHub pages,
including code coverage. I posted a question on Stack Overflow about it:

http://stackoverflow.com/questions/26028024/how-to-make-a-gulp-code-coverage-badge

* I want to be able to generate the coverage report after code coverage runs by any Node.js build system like
Gulp or Grunt, and be able to publish the badge on the README.md file through a Jenkins link;
* The badge displays appropriate colors for the badge.
 * Green: >= 80% overall coverage ![](http://img.shields.io/badge/coverage-93%-brightgreen.svg)
 * Yellow: 65% <= overall coverage < 80% ![](http://img.shields.io/badge/coverage-74%-yellow.svg)
 * Red: < 65% overall coverage ![](http://img.shields.io/badge/coverage-32%-red.svg)

The idea is to serve in the Node.js README for internal use in a GitHub enterprise machine along with
Jenkins.

Installation
=========

```
npm install --save-dev istanbul-cobertura-badger
```
Setup
=========

The following parameters are used:

* Cobertura file report: The XML-based cobertura file generated by Istanbul "cobertura" report.
* Destination Path: The path to the destination directory where the badge image will be created.
* Callback: The callback function after the file has been downloaded.
* Optional Thresholds: Adjust the thresholds if needed.

Cobertura Xml Report
--------

You can generate the cobertura XML report by adding a new report to the istanbul command as follows:

```
  --report cobertura
```

Take a look at this project's `package.json` for details.

The option to change in the code is as follows:

```js
  // Setting the default coverage file generated by istanbul cobertura report.
  opts.istanbulReportFile = opts.istanbulReportFile || "./coverage/cobertura-coverage.xml";
```

Destination Dir
-----

The destination directory will, by default, be the one from istanbul `$APP/coverage`.

```js
  // The default location for the destination being the coverage directory from istanbul.
  opts.destinationDir = opts.destinationDir || "./coverage/";
```

Note that the default directory will be used as `coverage/cobertura-coverage.xml`.

Adjusting Thresholds
---------

The overall threshold is computed by using the `line rate`, `branch rate` and `function rate`. That's the final value that defines 
`coverage result`. You can adjust the thresholds to properly generate the badges with appropriate colors.

```js
  // The thresholds to be used to give colors to the badge.
  opts.thresholds = {
    excellent: 90,
    good: 65
  };
```

* *Green*: coverage result >= opts.thresholds.excellent;
* *Yellow*: coverage result >= opts.thresholds.good;
* *Red*: coverage result < opts.thresholds.good.

Upon calling the callback function, the file `cobertura.svg` will be available in the destination path.

Examples
========

```js
var coberturaBadger = require('istanbul-cobertura-badger');

// Use the fixture that's without problems
var opts = {
  //badgeFileName: "cobertura", // No extension, Defaults to "coverage"
  destinationDir: __dirname, // REQUIRED PARAMETER!
  istanbulReportFile: path.resolve(__dirname, "coverage", "cobertura-coverage.xml"),
  thresholds: {
    // overall percent >= excellent, green badge
    excellent: 90,
    // overall percent < excellent and >= good, yellow badge
    good: 65
    // overall percent < good, red badge
  }
};

// Load the badge for the report$
badger(opts, function parsingResults(err, badgeStatus) {
  if (err) {
    console.log("An error occurred: " + err.message);
  }
  console.log("Badge successfully generated at " + badgeStatus.badgeFile.file);
  console.log(badgeStatus);
});
```

An example of a successful badge creation is as follows:

```js
{ overallPercent: 66,
  functionRate: 0.7368421052631579,
  lineRate: 0.8034,
  branchRate: 0.47369999999999995,
  url: 'http://img.shields.io/badge/coverage-66%-yellow.svg',
  badgeFile: { 
    method: 'GET',
    code: 200,
    file: '/home/mdesales/dev/github/intuit/istanbul-cobertura-badger/test/coverage.svg' 
  }
}
```

Gulp Build
=========

```js
gulp.task('test', function() {
  gulp.src('src/**/*.js')
    .pipe(istanbul()) // coverying files
    .on('finish', function () {
      gulp.src(['test/*.js'])
        .pipe(mocha({reporter: 'spec'})) // different reporters at http://visionmedia.github.io/mocha/#reporters
        .pipe(istanbul.writeReports({
          reporters: ['cobertura', 'text-summary', 'html'], // https://www.npmjs.org/package/gulp-istanbul#reporters
          reportOpts: { dir: './docs/tests' }
        }))
        .on('end', function() {

          var opts = {
            destinationDir: path.resolve(__dirname, "docs", "tests"),
            istanbulReportFile: path.resolve(__dirname, "docs", "tests", "cobertura-coverage.xml"),
          }
          coverageBadger(opts, function(err, results) {
            if (err) {
              console.log("An error occurred while generating the coverage badge" + err.message);
              process.exit(-1);
            }
            console.log("Badge generated successfully: " + results)
            process.exit(0);
           });

        });
    });
});
```

CLI
===

You can now use the CLI to create the badge for ANY XML Cobertura report created from Istanbul or Java applications.

CLI Options
------

The CLI prints the following help:

```
$ istanbul-cobertura-badger 

  Usage: cli [options]

  Generates a badge for a given Cobertura XML report

  Options:

    -h, --help                          output usage information
    -V, --version                       output the version number
    -f, --defaults                      Use the default values for all the input.
    -e, --excellentThreshold <n>       The threshold for green badges, where coverage >= -e
    -g, --goodThreshold <n>            The threshold for yellow badges, where -g <= coverage < -e  
    -b, --badgeFileName <badge>         The badge file name that will be saved.
    -r, --reportFile <report>           The istanbul cobertura XML file path.
    -d, --destinationDir <destination>  The directory where 'coverage.svg' will be generated at.
    -v, --verbose                       Prints the metadata for the command

  Examples:

    $ istanbul-cobertura-badger -e 90 -g 65 -r coverage/cobertura.xml -d coverage/
      * Green: coverage >= 90
      * Yellow: 65 <= coverage < 90
      * Red: coverage < 65
      * Created at the coverage directory from the given report.

    $ istanbul-cobertura-badger -e 80 -d /tmp/build
      * Green: coverage >= 80
      * Yellow: 65 <= coverage < 80
      * Red: coverage < 65
```

CLI Default Input
-----

Run the CLI using the default values defined above. Here's the example of running against this project.

```js
$ istanbul-cobertura-badger -f -v
{ overallPercent: 91,
  functionRate: 1,
  lineRate: 0.9309999999999999,
  branchRate: 0.8167,
  url: 'http://img.shields.io/badge/coverage-91%-brightgreen.svg',
  badgeFile: 
   { downloaded: true,
     filePath: '/home/mdesales/dev/github/intuit/istanbul-cobertura-badger/coverage/coverage.svg',
     size: 730 },
  color: 'brightgreen' }
Badge created at /home/mdesales/dev/github/intuit/istanbul-cobertura-badger/coverage/coverage.svg
```

CLI Simple Output
-----

Just use the simple command with some options.

```
$ istanbul-cobertura-badger -e 85 -g 70 -r test/fixture/istanbul-report.xml -d /tmp/
Badge created at /tmp/coverage.svg

```

CLI Verbose Output
-----

The overall information collected is also presented when using the verbose option.

```js
$ istanbul-cobertura-badger -e 85 -g 70 -r test/fixture/istanbul-report.xml -d /tmp/ -v
{ overallPercent: 66,
  functionRate: 0.7368421052631579,
  lineRate: 0.8034,
  branchRate: 0.47369999999999995,
  url: 'http://img.shields.io/badge/coverage-66%-red.svg',
  badgeFile: { downloaded: true, filePath: '/tmp/coverage.svg', size: 733 },
  color: 'red' }
Badge created at /tmp/coverage.svg
```

CLI Version
------

You can print the current version of the project by using the -V option. It uses the package.json#version as the value.

```
$ istanbul-cobertura-badger -V
1.0.0
```

Contributing
==============

We use the GitFlow branching model http://nvie.com/posts/a-successful-git-branching-model/.

1. Fork it
2. Create your feature branch (`git checkout -b feature/issue-444-Add-Rest-APIs origin/master --track`)
 * Adding the Jira ticket ID helps communicating where this feature is coming from.
3. Commit your changes (`git commit -am 'Fix #444: Add support to REST-APIs'`)
 * Adding "fix #444" will trigger a link to the GitHub issue #444.
4. Push to the branch (`git push feature/issue-444-Add-Rest-APIS`)
5. Create new Pull Request as indicated by this page or your forked repo.
