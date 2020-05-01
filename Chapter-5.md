# Define `resolve` method

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

var fetchResult = {
  // data: null,
  resolve: function(data) {
    this.callback(data);
  },
  done: function(callback) {
    // callback(this.data);
    this.callback = callback;
  }
};

fetchData(function (data) {
  // fetchResult.data = data;
  fetchResult.resolve(data);
});

module.exports = fetchResult;
```

We don’t need the `data` property anymore, and because the `fetchResult` is just a simple object, we need some way to share the `callback` between the `done` method and the `resolve` method which here we chose to set the `callback` as a method of the object (here `this` refers to the `fetchResult` object). You also can assign it to a variable in the parent scope of the object and also there might be some other ways).

And the `App.js` file remains the same as before.

In my opinion, at this point, we should refactor our code to have a better view of what we have done.
