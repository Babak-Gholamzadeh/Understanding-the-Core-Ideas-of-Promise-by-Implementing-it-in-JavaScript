# Register callbacks after resolving

One of our purposes in creating a reference was to enable us to access the resolved value of async functions from anywhere. At this moment, we can claim that we have achieved this goal.

Now is the time to work on another purpose. **We want to be able to access the resolved value of async functions at any time**.

If you test the program and try to run the `then` method after the async function is resolved, you will get an error that says: 

> TypeError: _onSuccess is not a function

And it’s absolutely right because once the `resolve/reject` function be called, it jumps to run the related callback.

In order to solve this problem, we need to find some way once the async function is resolved, first it should store the value and then check whether the corresponding callback has been set. If that has been set, it can be run, but if that hasn’t, it just does nothing and stays calm and ready.

On the other hand, whenever the `then` method be called, first it registers callbacks and then checks if the value has been resolved. If the value has been resolved already, the corresponding callback can be called, otherwise, nothing would happen.

So, for implementing this solution, because the execution of callbacks phase is common between both sides, it’s better to centralize that in a single function.

But this function is responsible for controlling the execution, it needs some information about the states.

Let’s see what is needed for executing. It must know if the value is ready or not and if it is, what the type of value is. It’s a resolved value or a rejected one. It also must know that if callbacks have been set or not.

As a result, it almost needs to know about these three things:

- Value *(it exists or not)*
- Value type *(it’s a resolved or a rejected)*
- Callbacks *(they have been set or not)*

In order to meet these requirements optimally, how about combining the first two cases?

We can set a state that is in **`pending`** mode by default, which means there isn’t any value yet and we are still waiting for the async function. And once the async function is done, depends on the situation that it is been resolved or rejected, we can set the relevant mode to the state (**`resolved`** or **`rejected`**).

```javascript
// ======================================
//   Inside async-controller.js module
// ======================================

function asyncController(borrowFunction) {
  var asyncHanlder = {};
  var _onSuccess = null;
  var _onError = null;
  var value = null;
  var state = 'PENDING';

  try {
    borrowFunction(resolve, reject);
  } catch (err) {
    reject(err);
  }

  function resolve(data) {
    // _onSuccess(data);
    value = data;
    state = 'RESOLVED';
    executeController();
  }

  function reject(err) {
    // _onError(err);
    value = err;
    state = 'REJECTED';
    executeController();
  }

  function executeController() {
    if(state === 'RESOLVED') {
      if(_onSuccess) {
        _onSuccess(value);
      }
    } else if (state === 'REJECTED') {
      if(_onError) {
        _onError(value);
      }
    }
  }

  asyncHanlder.then = function (onSuccess, onError) {
    _onSuccess = onSuccess;
    _onError = onError;
    executeController();
  }

  return asyncHanlder;
}

module.exports = asyncController;
```

Look at the code carefully. At the top, we defined two variables to store the value and the current state (which is `PENDING` state by default). And both of them only would be changed in the `resolve` or the `reject` function. After changing these variables the `executeController` will be called. It also can be called at the end of the `then` method.

Look at the `executeController`, it checks everything. And for now, you can use the `then` method any time you wish.

```javascript
// =================
//   Inside App.js
// =================

var asyncFetchData = require('./fetch.js');

asyncFetchData.then(
  onSuccess,
  onError
);

setTimeout(function() {
  asyncFetchData.then(
    onSuccess,
    onError
  );
}, 60_000 /* even after 1 minute */);

function onSuccess(data) {
  console.log('Result:', data); // output> Result: data is here!!!
}
function onError(err) {
  console.log('Error:', err); // output> Error: something is wrong!!!
}
```
