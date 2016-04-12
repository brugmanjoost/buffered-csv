# buffered-csv

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Usage](#usage)<br />
3.1 [Minimal](#minimal)<br />
3.2 [Buffering output](#buffering-output)<br />
3.3 [Dynamic filenames](#dynamic-filenames)<br />
3.4 [All options](#all-options)<br />
4. [Field handling](#field-handling)<br />
4.1 [Autodetect](#autodetect)<br />
4.2 [Prior specification](#prior-specification)<br />
5. [API](#api)<br />
5.1 [Class: buffered-csv.File](#class-buffered-csvfile)<br />
&nbsp;&nbsp;&nbsp;&nbsp;5.1.1 [Event: 'data'](#event-data)<br />
&nbsp;&nbsp;&nbsp;&nbsp;5.1.2 [Event: 'error'](#event-error)<br />
&nbsp;&nbsp;&nbsp;&nbsp;5.1.3 [file.add(data)](#fileadddata)<br />
&nbsp;&nbsp;&nbsp;&nbsp;5.1.4 [file.complete()](#filecomplete)<br />
&nbsp;&nbsp;&nbsp;&nbsp;5.1.5 [file.flush()](#fileflush)<br />
6. [To do](#to-do)

## Introduction
Generation of csv files often involves appending the generated csv file line by line as the data is generated. Some use cases benefit from storing the data in memory until a threshold is reached upon which all lines in memory are appended to the file at once. This module does just that:
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
The following displays the bare minimum usage using all defaults and no buffering. Lines are written to file line-by-line, similar to classic CSV generation.

```javascript
const csv = require('buffered-csv');
var file = new csv.File({
    path: 'celebrities.csv'
});
file.add({
    Name: 'Albert Einstein',
    Expertise: 'Relativity'
});
file.add({
    Name: 'Galileo Galilei',
    Expertise: 'Gravity',
    Birthyear: 1564
});
file.add({
    Name: 'Shen Kuo',
    Expertise: 'Astronomy',
});
file.complete();
```

### Buffering output
Turning on buffering only requires passing flushing parameters to the constructor, like so:
```javascript
var file = new csv.File({
    path: 'celebrities.csv',
    flushInterval: 5000,
    flushLines: 50
});
```
This flushes data from memory to file:
- every 5000 milliseconds.
- or as soon as the buffer contains 50 lines.
- or as soon as file.close() is called.

In our [minimal usage](#minimal) example we only add three lines, less than the specified flushLines of 50. We also add them well within 5000ms. If we used these settings in the example, the data would be written to file no earlier then when file.complete() is called.

### Dynamic filenames

The location of the output file is given through the *path* option and may be specified as a string or as a callback function. This allows for the use case where the writes are not only buffered, but also distributed across different writes:

```javascript
var file = new csv.File({
    path: function() {
        return 'celebrities_' + (new Date()).getTime() + '.csv';
    },
    flushInterval: 5000,
    flushLines: 50
});
```

If chosen to write headers, then headers are written to each file. Buffered-csv maintains header information and field layout across writes and files. Concatenating the resulting output files into a single csv will generate a valid file.

### All options
The following is a full list of all options that may be passed to the constructor:

|Option             |Default       |Description                                                                                                                                                                     |
|-------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**encoding**       |'utf-8'.      |Character encoding for the output file.                                                                                                                                         |
|**delimeter**      |','           |The character used to seperate values on a single line.                                                                                                                         |
|**quote**          |'"'           |The character used to enclose string values.                                                                                                                                    |
|**escape**         |'\\'          |The escape character to use if the quote character is present in a string value.                                                                                                |
|**nullValue**      |'NULL'        |A string to insert for fields that have an *undefined* value. This string is inserted verbatim in the csv output without any quoting or escaping.                               |
|**eol**            |System default|A string to designate the end of line. Typically '\n' or '\r\n'.                                                                                                                |
|**headers**        |true          |True if the first line of each output file must contain header information. False otherwise.                                                                                    |
|**overwrite**      |true          |True if the output files must be overwritten upon the first write to that file. If dynamic filenames are used each file will be overwritten upon the first write to that file.  |
|**flushInterval**  |0 (disabled)  |The contents of the csv buffer are written to file after each specified interval in milliseconds. Ignored if 0. *                                                               |
|**flushLines**     |0 (disabled)  |The contents of the csv buffer are written to file as soon as the buffer contains the specified number of lines. Ignored if 0. *                                                |
|**fields**         |{}            |A specification of fields and how to handle them, in the following format: { &lt;name&gt;: { type: 'string'\|'number' } }


**Note:** If both *flushInterval* and *flushLines* are 0 buffered-csv operates in classic mode. Lines are written to file immediately when added.

## Field handling

buffered-csv tracks field order and field typing across all writes and all files. If data is added as an array buffered-csv will always assume all entries in the array match up with any previously established field order. If data is added as a key/value map then buffered-csv will ensure fields will always be sent to file in the same order.

## Autodetect
If data is added as a key/value map previously unknown fields will automatically be detected and assumed to be string (quoted) types. This works well for many simple csv scenario's but there are drawbacks:

In our [minimal usage](#minimal) example a Birthyear...
* ... is not specified for Albert Einstein. His csv line will have two fields.
* ... is specified for Galileo Galilei. His csv line will have three fields.
* ... is not specified for Shen Kuo. His csv line will have three fields with a *nullValue* for *Birthyear.*

Without buffering data for Albert Einstein is sent to file immediately, which requires immmediate generation of a header line based on fields known so far. This excludes the *Birthyear* field which is only introduced later.

In our [buffering output](#buffering-output) example data is only sent to file after Galileo Galilei has been added to the buffer. As a consequence...
* ... the header line will include the *Birthyear* field.
* ... Albert Einstein's csv line will have  three fields with a nullValue for Birthyear.

With dynamic filenames field information is maintained across all files. If during the 5th write on the 3rd file a field is added, it will show up in the headers of the 6th file and any subsequent files. It will not show in the headers of files 1 through 2. It will also not show in the headers of file 3 (it would have to be detected on the 1st write of file 3).

## Prior specification

Prior specification of fields solves the drawbacks of autodetect at the expense of a requirement for prior knowledge. Fields may be specified to the constructor like so:

```javascript
var file = new csv.File({
    path: 'celebrities.csv',
    fields: {
        Expertise: {
            type: 'string'
        },
        Name: {
            type: 'string'
        },
        Birthyear: {
            type: 'number'
        }
    }
});
```

In this case *Birthyear* will show up in all headerlines in all files. Additionally this method specifies the field type, currently either 'string' (quoted) or 'number' (non-quoted). Finally, this also sets the field order so that *Expertise* will be the first field listed on each line.

Using pior specification does not disable autodetect and does not enable an error mechanism if unkown field names are added.

## API

### Class: buffered-csv.File

#### Event: 'data'
'data' is emitted each time the contents of the buffer is written to file. *path* contains the actual file targeted and *csv* the actual content written to the file.

```javascript
file.on('data', function(path, csv) {
}
```

#### Event: 'error'
'error' is emitted if writing to a file failed. buffered-csv leaves handling this error up to you. One way to handle it could be to perform a rewrite to the file yourself or saving the contents at an alternative location.

```javascript
file.on('error', function(err, path, csv) {
}
```
An 'error' event is always preceeded by a 'data' event.

#### file.add(data) 
Adds a line of data to the csv file. The line is added to the buffer initially and once the flush thresholds are met, the data is sent to file.

If *data* is an array then the data is interpreted as order-first. Array items are interpreted as fields according to their position in the array. This influences their representation in the file (e.g. quoted or not).

If *data* is a key/value map then the data is interpreted as name-first. Each key is interpreted as a field name. Data is sent to the buffer with a field order that matches any earlier established order of fields. Previously unseen keys are autodiscovered as headers.

#### file.complete() 

Finishes generation of the csv file by flushing any remaining data in the buffer and terminating any running timers.

#### file.flush()

Explicitly flushes the buffer to file.

## To do
There are some items left to enhance buffered-csv, namely:

 - **Custom field support:** buffered-csv internally supports fields of any type through a mapping called *convertors*. Currently *string* and *number* are the only ones implemented where string is quoted and number is not. It would be chique to expose an API to allow adding custom items to this list, e.g. to convert a *Date* object to a YYYY-MM-DD representation.
 - **Timeout flush:** Initiate a timeout once the first row enters the buffer. A flush is then initiated when the timeout completes, regardless of buffer size. If a new row enters the buffer after this the cycle repeats. This is an enhanced version of Interval flush that maximizes the amount of data written in a single flush.

This is an open souce project. Go ahead if you feel obliged.
 
