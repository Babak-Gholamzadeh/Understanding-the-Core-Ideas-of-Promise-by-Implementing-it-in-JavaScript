# Start with a simple variable

Let’s start with a stupid idea to implement. Assume there is a function named `fetchData` that fetches some data from some resource asynchronously. We create a file named `fetch.js` and call the `fetchData` function inside it and at the end export the result.

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

var fetchResult = undefined;

fetchData(function (data) {
  fetchResult = data;
});

module.exports = fetchResult;
```

Let’s import this module in another file to see what exactly happens.

```javascript
// =================
//   Inside App.js
// =================
var fetchResult = require('./fetch.js');

console.log(fetchResult); // output> undefined;

setTimeout(function() {
  console.log(fetchResult); // output> undefined;
}, 1000); // assume fetchResult takes data in less than 1 sec
```

At the first line, the `require` function be called. The codes inside the `fetch.js` be run and the value of `fetchResult` variable be exported. So at the time of exporting, the callback of `fetchData` hasn’t run yet and the data hasn’t been assigned to `fetchResult` variable; which means the value of `undefined` - that was assigned - would be exported.

The returned value of the `require` function would be put into the `fetchResult` variable in `App.js` file (which is the `undefined` value) and even when we try to access the variable few moments later (inside the callback of `setTimeout` function) (which is assumed that be called after resolving the `fetchData`), it still displays the `undefined` value.

The key point that causes this result is that the initial value [`undefined`] is a primitive type.

## A brief explanation about primitive and non-primitive types in JavaScript

*Note: This is just a reminder. Feel free to skip it if you already know about it.*

Primitive types in JavaScript, which are all data types except object types like `object`, `function`, `array`, `map`, `set`, etc, would be copied by value (in abstract level). In other words, if you assign a primitive value to a variable, then assign that variable to another variable, the second variable will get a copy of that value not a reference to that value. I know this explanation made you more confused, so to understand it better look at the example below:

```javascript
var a = 3;
var b = a;
a = a + 1;

console.log(a); // output> 4
console.log(b); // output> [still] 3
```

As you see the value inside of the `b` is not changed after we changed the value inside of the `a` (because the type of this value is primitive).

Let’s see another example of non-primitive types which be copied by reference:

```javascript
var a = {
  value: 3
};
var b = a;
a.value = a.value + 1;

console.log(a); // output> { value: 4 }
console.log(b); // output> { value: 4 }
```

Great! Now a light bulb might go over your head and thinking this is what we need to access the resolved value of an async function.

OK, let’s back to our codes and try to use an object instead of a primitive value.
