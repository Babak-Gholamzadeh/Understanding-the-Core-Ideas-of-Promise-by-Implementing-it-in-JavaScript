# Send the callback to the object

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

var fetchResult = {
  data: null,
  done: function(callback) {
    callback(this.data);
  }
};

fetchData(function (data) {
  fetchResult.data = data;
  // fetchResult.done(data);
});

module.exports = fetchResult;
```

What if we define a method named `done` in initialization that takes our function and calls it by passing the resolved value to it as an argument?

```javascript
// =================
//   Inside App.js
// =================

var fetchResult = require('./fetch.js');

// fetchResult.done = function(data) {
//   console.log(data);
// };

fetchResult.done(function(data) {
  console.log(data); // output> null
});
```

And in the `App.js` we don’t manipulate the object and just pass a callback to the `done` method.

Good! We didn’t break any rule. But wait! Our program doesn’t work! How do we suppose to call the `done` method inside the callback of the async function? It needs a callback as its parameter, it’s kind doesn’t make sense at all. We came to a deadlock. It’s okay guys! It’s the life, sometimes works and sometimes doesn’t.

Let's take a closer look at the situation. So far, we sent a callback to the `done` method that we are not able to call it. How about adding another method to the object that calls our callback? This idea is like that, you register your callback with the `done` method and call it with another method. Let’s name the other method `resolve` because it’s going to be called when the async function is resolved.