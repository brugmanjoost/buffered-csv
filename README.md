buffered-csv
============

1. [Introduction](#Introduction)
2. [Installation](#Installation)
2. [How It Works](#how-it-works)
3. [API Documentation](#api-documentation)

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

```javascript
const csv = require('buffered-csv');

var file = new csv.File({
  'encoding':       'utf8',
  'delimeter':      ',',
  'quote':          '"',
  'escape':         '\',
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
