buffered-csv
============

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Usage](#usage)

## Introduction
Generation of csv files is straightforward. Often this involves appending the generated csv file line by line as the data is generated. Some use cases benefit from storing the data in memory until a threshold is reached upon which all lines in memory are appended to the file at once. This module does just that:
- Buffering generated csv lines in memory:
  - Writing to file only after the buffer acccumulates a specified number of lines.
  - Writing to file at preset intervals.
  - Or traditionally writing line-by-line as data is generated.
- Dynamic filename generation for each write.
- Automatic detection of field headers based on buffer data on file creation.

## Installation
```sh
$ npm install buffered-csv
```

## Usage

### Minimal
The following displays the bare minimum usage using all defaults and no buffering. Lines are written to file line-by-line.

```javascript
const csv = require('buffered-csv');
var file = new csv.File();
file.open('celebrities.csv');
file.add({
  Name: 'Albert Einstein',
  Expertise: 'Relativity',
});
file.add({
  Name: 'Galileo Galilei',
  Expertise: 'Gravity',
});
file.close();
```

### Buffered output
Turning on buffering only requires passing flushing parameters to the constructor, like so:
```javascript
var file = new csv.File({
  'flushInterval':  5000,
  'flushLines':     3
});
```
This flushes data from memory to file:
- every 5 seconds.
- or as soon as the buffer contains three lines.
- or as soon as file.close() is called.

### All options
The following is a full list of all options that may be passed to the constructor:

|option|description|
|---------------|---------------------------------------------------------|
|encoding       |Defaults to 'utf-8'. Specified encoding for the output file.       |
|delimeter      |Defaults to ','|
|quote          |Defaults to '"'|
|escape         |Defaults to '\\'|
|nullValue      |Defaults to 'NULL'|
|eol            |Defaults to the system default.|
|headers        |Defaults to true.|
|overwrite      |Defaults to true.|
|flushInterval  |Defaults to 0.|
|flushLines     |Defaults to 0.|
|fields         |Defaults to {}|













```javascript






var file = new csv.File({
  'encoding':       'utf8',
  'delimeter':      ',',
  'quote':          '"',
  'escape':         '\\',
  'nullValue':      'NULL',
  'eol':            require('os').EOL,
  'headers':        true,
  'overwrite':      true,
  'fields':         {},
  'flushInterval':  0,
  'flushLines':     0
});
file.open('report.csv');

file.close();
```
