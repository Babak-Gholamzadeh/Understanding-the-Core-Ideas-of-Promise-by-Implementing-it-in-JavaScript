# Initialize with an object instead of undefined

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

// var fetchResult = undefined;
var fetchResult = {};

fetchData(function (data) {
  // fetchResult = data;
  fetchResult.data = data;
});

module.exports = fetchResult;
```

*Note: I commented the previous codes that you could see the changes.*

Now we defined an object and export a **reference** to that object.

```javascript
// =================
//   Inside App.js
// =================
var fetchResult = require('./fetch.js');

console.log(fetchResult); // output> {};

setTimeout(function() {
  console.log(fetchResult); // output> { data: data is here!!! };
}, 1000); // assume fetchResult takes data in less than 1 sec
```

At the point that we immediately tried to access the resolved value, we got nothing except an empty object. But after a few moments that the async function is resolved, we are completely able to access the value. Great!

It’s right that we could access the resolved value outside of the callback, but there is still another problem. We used a `setTimeout` function (with a constant value as its time) to check the object whether the data is resolved.

How can we solve this problem? There is no specific time that we could calculate it somehow to know how much we must wait, it doesn’t make sense. We need something to be triggered once the async function is resolved to inform us that now we can read the value.

How about to call a method of our object inside the callback to notify us in another module that the value is ready. For implementing this approach we need a conventional name for the method. Because this method must be defined in another place and be called inside the callback, So the callback must know the name of the method that you defined in somewhere else.

Let’s use the name `done` as the conventional name for the method.
