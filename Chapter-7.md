# Handle the errors

So far, we are on the right track. But we want our code to be more maintainable, we need to have more control to handle the errors that might occur.

After we refactored the code, now it’s much easier to add new features to it. The logic of handling error is similar to resolving data. We are kind of going the same path but instead of resolving, we reject.

To clarify the subject, this time we ought to start tweaking from the `App.js` to see what we need to handle the errors.

```javascript
// =================
//   Inside App.js
// =================

var fetchResult = require('./fetch.js');

// fetchResult.done(function (data) {
//   console.log(data);
// });

// 'then' method is just the same as 'done' method
// but takes two callbacks instead of one
fetchResult.then(
  // if data be fetched successfuly, then 'onSuccess' would be called
  onSuccess,
  // if any error occurs, then 'onError' would be called
  onError
);

function onSuccess(data) {
  console.log('Result:', data); // output> Result: data is here!!!
}
function onError(err) {
  console.log('Error:', err); // output> Error: something is wrong!!!
}
```

First of all, we changed the name of the `done` method to `then`, because for this purpose it makes more sense.

After that we pass two callbacks to the object to be registered instead of one. The first one would be called when the async function is resolved and the second one is for a rejection or any sort of error occurring.

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

function asyncController(asyncFunction) {
  var asyncHanlder = {};
  // var _callback = null;
  var _onSuccess = null;
  var _onError = null;

  asyncFunction(function (data) {
    // resolve(data);
    if (data) {
      resolve(data);
    } else {
      reject('something is wrong!!!');
    }
  });

  function resolve(data) {
    // _callback(data);
    _onSuccess(data);
  }
  function reject(err) {
    _onError(err);
  }

  // asyncHanlder.done = function (callback) {
  //   _callback = callback;
  // }
  asyncHanlder.then = function (onSuccess, onError) {
    _onSuccess = onSuccess;
    _onError = onError;
  }

  return asyncHanlder;
}

var asyncHanlder = asyncController(fetchData);
module.exports = asyncHanlder;
```

In the first lines, we used two variables instead of one single `_callback` for referring to our new callbacks.
The `then` method in the bottom section, register the callbacks to their placeholders.

In the `asyncFunction` that previously we just called the `resolve` function, now we also can call the `reject` function on the situations that we believe something is wrong and it needs to be handled.

In the middle part that the `resolve` and the `reject` functions be defined, you can see each of them calls the corresponding callback.

*Note: From here onwards, code will grow and get more complex on each step. So try to read it patiently and make sure you understand it before jumping to the next step.*
