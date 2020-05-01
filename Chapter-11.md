# Eliminate nesting of thenable object

So far so good. We have implemented chaining and now our program is no longer needed to put everything inside one `then` method, rather it can chain the steps after each other.

The only weakness that our chaining pattern has that it cannot chain several async functions and run them respectively.

Let’s see our final desire:

```javascript
var asyncFetchData = require('./fetch.js');

asyncFetchData
  .then(function (result) {
    console.log(result); // output> data is here!
    // saveData is an async function
    // and the returned value of it is a thenable object (like asyncFetchData)
    return saveData(result);
  })
  .then(function (result) {
    console.log(result); // output> data saved successfuly!
  });
```

You can see that the returned value of the `saveData` function is similar to the `asyncFetchData` object. And in these kinds of situations, I mean when we realize that the returned value of the callback is thenable, afterward the next `then` method which comes after it, should acts as its own `then` method.

In the example above, the second `then` method is exactly the `then` method of the `saveData`.

Before start coding, let’s review the workflow of our program again to see what exactly is going on.

In the first `then` method, when its callback (i.e. the `onSuccess`) be run, the returned value of that – which is a thenable object – will be sent to the `resolve` of the next `asyncController`.

In the `resolve` function we assign this thenable object to the `value` variable and then call the `executeController` and after that, this object would be sent to the `_onSuccess` function and also over there it will be passed to the `onSuccess` callback that is the same callback in the second `then` method.

Finally, if in the callback of the second `then` method we print the `result` variable, we can see that it is a thenable object. And if you want to access the result of that, you must use a nested `then` method.

```javascript
var asyncFetchData = require('./fetch.js');

asyncFetchData
  .then(function (result) {
    console.log(result); // output> data is here!
    return saveData(result);
  })
  .then(function (result) {

    result.then(function(nextResult) {
      console.log(nextResult); // output> data saved successfuly!
    });

  });
```

It’s definitely not what we want. We don’t want to get back to nesting stuff.

Fortunately, this issue is not a big deal and just two thenable objects are wrapped inside of each other.

The only thing that we need to do is, before sending the resolved value to the `_onSuccess` function, just unwrap it and send its pure value.

In order to cover this issue, in the `resolve` and the `reject` functions, before doing anything, first, we must check if the value is thenable, then unwrap it and continue the process.

For recognizing thenable objects, we should write a simple function to do this job for us.

By [the definition of thenable object in Promise/A+ specification](https://promisesaplus.com/#point-7):
> `thenable` is an object or function that defines a `then` method.

OK! Now we totally know how to check a thenable object. Let’s write its function.

```javascript
function isThenable(value) {
  return (

    value !== null &&

    (
      typeof value === 'object' ||
      typeof value === 'function'
    ) &&

    typeof value.then === 'function'

  );
}
```

Because in JavaScript the type of `null` value is also object, first we should make sure that the value is not `null`.

Don’t worry if the value is an `array`, actually, it doesn’t matter. Because we have to be sure that the value is able to have property and method. So there is no difference if it is a plain `object`, `array`, `function`, or even `map` or `set` (all of them are kind of object).

In my opinion, you don’t even have to check if the value is an object or something similar to that, but be careful because if the value be `null` or `undefined` and you try to access a method on them (like in the last condition), you will get an error.

People usually do it in this way to be more explicit. Because explicitly makes your code more readable.

After recognizing thenable objects - for unwrapping them - we just need to call their `then` method and pass the `resolve` and the `reject` functions to it as its callbacks. By doing that, we extract the pure value of it and then the `resolve` or the `reject` function - depends on the situation - will be called.

```javascript
function resolve(data) {
  if (isThenable(data)) {
    return data.then(resolve, reject);
  }

  value = data;
  state = 'RESOLVED';
  executeController();
}


function reject(err) {
  if (isThenable(err)) {
    return err.then(resolve, reject);
  }

  value = err;
  state = 'REJECTED';
  executeController();
}
```

Oh! These functions look ugly now! Let’s make them a little bit cleaner and DRY.

```javascript
var resolve = applyNewState('RESOLVED');
var reject = applyNewState('REJECTED');

function applyNewState(newState) {
  return function (newValue) {

    if (isThenable(newValue)) {
      return newValue.then(resolve, reject);
    }

    value = newValue;
    state = newState;
    executeController();

  }
}
```

I’m so sorry if this refactoring seems a little complicated (this way the first thing that came to my mind). But don’t worry, stay with me, I am going to explain it.

When we call the `applyNewState` and pass a new state to it, it will store the new state in the `newState` variable and return another function.

Thanks again to closure. We still can access the `newState` variable inside the returned function.

Now the `resolve` and the `reject` variables both are new functions, except that they don’t have to assign their state to the `state` variable explicitly because that has already be set in the `newState` variable.

The rest of the codes are simple and the first thing they do is to solve the problem with thenable objects.

Until here, our chainable pattern works properly. It works with synchronous and asynchronous codes. But let’s add a tiny improvement to the `executeController` function to treat both synch and async codes the same.

We want that any kind of code inside the callbacks of the `then` methods always be run asynchronously.

The easiest way to achieve that is to put the content of the `executeController` inside a `setTimeout` function.

```javascript
function executeController() {

  setTimeout(function () {

    if (state === 'RESOLVED') {
      if (_onSuccess) {
        _onSuccess(value);
      }
    } else if (state === 'REJECTED') {
      if (_onError) {
        _onError(value);
      }
    }

  }, 0);

}
```

Because we just want to run the code asynchronously (not after a while), we put `0 ms` there.
