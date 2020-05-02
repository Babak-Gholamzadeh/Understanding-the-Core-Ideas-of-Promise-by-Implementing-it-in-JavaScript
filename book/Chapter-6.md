# Refactor the code

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

function asyncController(asyncFunction) {
  var asyncHanlder = {};
  var _callback = null;

  asyncFunction(function (data) {
    resolve(data);
  });

  function resolve(data) {
    _callback(data);
  }

  asyncHanlder.done = function (callback) {
    _callback = callback;
  }

  return asyncHanlder;
}

var asyncHanlder = asyncController(fetchData);
module.exports = asyncHanlder;
```

Wow! It sounds that this refactoring was really helpful. We create a function named `asyncController` that takes an async function parameter and returns an object named `asyncHanlder` which can be used to register a callback by its `done` method. We encapsulated the operation inside a function which causes to minimize the exposure. Also the `App.js` is still untouched.

*Recommendation: Make sure you have completely understood until here, then go further.*
