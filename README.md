#js-measure (WIP)

Measure the raw, minified, and gzipped size of javascript packages including a breakdown of dependency contributions to the final size.  The CLI can also be used to measure packages in npm so they can be evaluated for inclusion in your project.

This module draws a lot of inspiration from the excellent [disc](https://github.com/hughsk/disc) project, and was created in order to develop a lower level service layer which would:

A) Provide a programmatic API and CLI
B) Be a viable candidate for refactoring into the disc tool.

# Installation

```sh
npm install --global js-measure
```

# Usage

There are three distinct measurement methods, described below.  Regardless of which technique is used, the output reporting format will be the same.

## Direct Bundle Measurement

Measure final browser-packaged bundle.  Dependencies will be measured if and only if full paths are included in require statements.  This method uses some heuristics to determine the baseline bundling costs, and will be less accurate than entry point measurement as a result.

```sh
js-measure -f path/to/final-bundle.js
```


## Entry Point Measurement

Measure dependencies using entry point for bundling, which by default will use browserify with full path resolution.  A custom bundling command can also be provided, if a bundler other than browserify is being used, or you wish to specify other browserify options.

Entry measurement provides greater accuracy since the bundling cost can be measured empirically by bundling an empty entry point.  The downside is that it will add bundling time to your build process.  This cost can be mitigated by capturing output with the `-o` option.

```sh
# Default: browserify
js-measure -e path/to/entry.js
# Easy addition of harmony flag
js-measure --harmony -e path/to/entry.js

# Specify bundling command.  The entry point will be appended to this command.
js-measure -e path/to/entry.js -b 'browserify --full-paths -e' -o bundle.js

# Specify bundling and minification.
js-measure -e path/to/entry.js -b 'browserify --full-paths -e' -m 'uglify-js' -o bundle-min.js
```

## Module Measurement

You can measure a module that you have not yet imported into your project.  The module will be installed in a temporary directory, bundled with browserify (or specified option) and measured, along with its sub-dependencies.

```sh
js-measure -m react
```

# API


## module

### module.measureFile

`ReportPromise module.measureFile(String fileName, [MeasureOptions opt])`

Perform measurement on a file or Readable stream.

Usage:

```js
require('js-measure').file('file.js').then(function(report) {
  console.debug(report);
});

```

### module.measureEntry

`ReportPromise module.measureEntry(String fileName, [MeasureOptions opt])`



## ReportPromise

ReportPromise is a Promise which will resolve with a Report. Additionally there a couple mixin functions providing access to the underlying output streams: `reportStream` and `outputFileStream`


### ReportPromise.reportStream

`ReadableStream reportPromise.reportStream()`

Returns a ReadableStream for the formatted report, enabling the following syntax:

```js
require('js-measure').measureFile('file.js').reportStream().pipe(myOutputStream);
```

### ReportPromise.outputFileStream

`ReadableStream reportPromise.outputFileStream()`

Returns a ReadableStream for the final output file of the operation.  For `module.measureFile` calls, this is simply a stream for the file under analysis, and for other measurement calls, this will be a readable stream for the packaged/browserified file.

```js
var reportPromise = require('js-measure').measureEntry('./src/entry.js');
reportPromise.outputFileStream().pipe(myOutputStream);
reportProimse.reportStream().pipe(myReportStream);
```

## Report

Object containing final report details


### Report.getString()

Get a formatted report as a string.

### Report.stream()

Get an readable stream from the report.

## SizeMetric

Object containing size-related measurements in useful formats.

# TO BE CONTINUED...
