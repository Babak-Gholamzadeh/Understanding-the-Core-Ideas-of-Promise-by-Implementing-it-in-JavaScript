# Define `done` method on the object

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

var fetchResult = {};

fetchData(function (data) {
  // fetchResult.data = data;
  fetchResult.done(data);
});

module.exports = fetchResult;
```

Here we call the `done` method of the object and also pass the resolved value to it as an argument (we don’t need to set the value in a property anymore).

```javascript
// =================
//   Inside App.js
// =================

var fetchResult = require('./fetch.js');

// setTimeout(function() {
//   console.log(fetchResult);
// }, 1000);

fetchResult.done = function(data) {
  console.log(data); // output> data is here!!!
};
```

And inside the `App.js` we defined the `done` method. Awesome! Now we know when the data is ready and It’s a good achievement.

But on the other hand, we still need to improve it, because it’s not a good idea that another user to change our object and add/remove some property or method to/from it.

We need to send our function to this object somehow. Let’s tweak our code.

