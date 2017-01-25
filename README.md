[![NPM](https://nodei.co/npm/quick-pivot.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/quick-pivot/)

[![npm version](https://badge.fury.io/js/quick-pivot.svg)](https://www.npmjs.com/package/quick-pivot)
[![Build Status](https://img.shields.io/travis/pat310/quick-pivot.svg)](https://travis-ci.org/pat310/quick-pivot)
[![Coverage Status](https://coveralls.io/repos/github/pat310/quick-pivot/badge.svg?branch=addingCoveralls)](https://coveralls.io/github/pat310/quick-pivot?branch=addingCoveralls)
[![Code Climate](https://codeclimate.com/github/pat310/quick-pivot/badges/gpa.svg)](https://codeclimate.com/github/pat310/quick-pivot)

## What it does
Say you have this example data set:<br>
![example data](/screenshots/ss1.png)

With this tool you can pivot the data given a particular row and column category:<br>
![example pivot 1](/screenshots/ss2.png)

Or given multiple rows and a column category:<br>
![example pivot 2](/screenshots/ss3.png)

Or multiple columns and a row category:<br>
![example pivot 3](/screenshots/ss4.png)

Or any combination of rows and/or columns

## Example use
Install with npm:
`npm install --save quick-pivot`


```js
var pivot = require('quick-pivot');

var dataArray = [
 ['name', 'gender', 'house', 'age'],
 ['Jon', 'm', 'Stark', 14],
 ['Arya', 'f', 'Stark', 10],
 ['Cersei', 'f', 'Baratheon', 38],
 ['Tywin', 'm', 'Lannister', 67],
 ['Tyrion', 'm', 'Lannister', 34],
 ['Joffrey', 'm', 'Baratheon', 18],
 ['Bran', 'm', 'Stark', 8],
 ['Jaime', 'm', 'Lannister', 32],
 ['Sansa', 'f', 'Stark', 12]
];

var rowsToPivot = ['name'];
var colsToPivot = ['house', 'gender'];
var aggregationCategory = 'age';
var aggregationType = 'sum';

var pivotedData = pivot(dataArray, rowsToPivot, colsToPivot, aggregationCategory, aggregationType);

console.log(pivotedData);
```

console logs:
```js
{ table:
 [ [ 'sum age', 'Stark', 'Stark', 'Baratheon', 'Baratheon', 'Lannister' ],
   [ 'sum age', 'm', 'f', 'f', 'm', 'm' ],
   [ 'Jon', 14, '', '', '', '' ],
   [ 'Arya', '', 10, '', '', '' ],
   [ 'Cersei', '', '', 38, '', '' ],
   [ 'Tywin', '', '', '', '', 67 ],
   [ 'Tyrion', '', '', '', '', 34 ],
   [ 'Joffrey', '', '', '', 18, '' ],
   [ 'Bran', 8, '', '', '', '' ],
   [ 'Jaime', '', '', '', '', 32 ],
   [ 'Sansa', '', 12, '', '', '' ] ],
rawData:
 [ [ 'Jon', [Object], '', '', '', '' ],
   [ 'Arya', '', [Object], '', '', '' ],
   [ 'Cersei', '', '', [Object], '', '' ],
   [ 'Tywin', '', '', '', '', [Object] ],
   [ 'Tyrion', '', '', '', '', [Object] ],
   [ 'Joffrey', '', '', '', [Object], '' ],
   [ 'Bran', [Object], '', '', '', '' ],
   [ 'Jaime', '', '', '', '', [Object] ],
   [ 'Sansa', '', [Object], '', '', '' ] ] }
```

## API
### Return value
`quick-pivot` returns an object with keys `table` and `rawData`.  `table` is an array of arrays containing the result of the pivot. `rawData` is an array of arrays but rather than returning the result of the pivot, it returns the data points that make up the result in each corresponding index.  This allows the user to determine what data made up the pivoted value.

### Syntax

```js
var pivot = require('quick-pivot');

pivot(dataArray, rows, columns, [accumulationCategory or CBfunction], [accumulationType or initialValue], rowHeader); 
```

#### First way to use it:
* `dataArray` **required** is one of the following:
 * array of arrays ( the array in first index is assumed to be your headers, see the example above)
 * array of objects (the keys of each object are the headers)
 * a single array (a single column of data where the first element is the header)
* `rows` is an array of strings (the rows you want to pivot on) or an empty array **required**
* `columns` is an array of strings (the columns you want to pivot on) or an empty array **required**
* `accumulationCategory` is a string (the category you want to accumulate values for) **required**
* `accumulationType` is an enumerated string - either `'sum'` or `'count'` (the type of accumulation you want to perform). If no type is selected, `'count'` is chosen by default 
* `rowHeader` is a string (this value will appear above the rows)

#### Second way to use it:
Parameters are the same as the first except for two, `accumulationCategory` and `accumulationType`. Instead of `accumulationCategory` and `accumulationType`, you can use the following:
* `CBfunction` is a callback function that receives four parameters `CBfunction(acc, curr, index, arr)` where `acc` is an accumulation value, `curr` is the current element being processed, `index` is the index of the current element being processed and `arr` is the array that is being acted on. This function must return the accumulation value (this is very similar to javascript's `.reduce`) **required**
* `initialValue` is the starting value for the callback function. If no starting value is selected, `0` is used by default.

###### Example with callback function

```js
function cbFunc(acc, curr, index, arr){
  acc += curr.age;
  if(index === arr.length - 1) return acc / arr.length;
  return acc;
}
var table = pivot(dataArray, ['gender'], ['house'], cbFunc, 0, 'average age');

console.log(table);
/*
{ table:
 [ [ 'average age', 'Stark', 'Baratheon', 'Lannister' ],
   [ 'm', 11, 18, 44.333333333333336 ],
   [ 'f', 11, 38, '' ] ],
rawData:
 [ [ 'm', [Object], [Object], [Object] ],
   [ 'f', [Object], [Object], '' ] ] }
*/
```
