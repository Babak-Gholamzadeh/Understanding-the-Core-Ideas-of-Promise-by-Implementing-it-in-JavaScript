# Modularize the code

As I said, because our program is going to get more complex, for controlling it we need to somehow refactor the code and separate its logic.

By breaking the code into smaller chunks and put them in separate files, we are actually modularizing the code. But this action must be done logically. This means that we should investigate the code more carefully to figure out which parts can act separately and independently and even they are not related to each other and have a different purpose.

By looking at the `fetch.js` we see there is a function named `asyncController` that its purpose is to control async functions in general (which means it can control any async function). But when we dive into this function we see some operation that is specific to the `fetchData` async function. This piece of code prevents the `asyncController` to act independently. And also we can see that all of these logics are put in a single module named `fetch.js` that doesn’t make sense either.

How about extracting the codes that are just relevant to the `asyncController` and put them in a separate file named `async-controller.js` that even can be reused for every other async function?

After doing that, we can import the `asyncController` in any other module to use it for controlling asynchronism.

Now we found out that we should separate the `asyncController` from the `fetchData`, But these two functions stuck together. How do we suppose to do that?

At this point, we need to think about refactoring the code to find some way to pull the `asyncFunction` out of the `asyncController`.
Let’s take a closer look again:

```javascript
asyncFunction(function (data) {
  if (data) {
    resolve(data);
  } else {
    reject('something is wrong!!!');
  }
});
```

The `asyncFunction` (as a parameter of the `asyncController`) would come here just to use the `resolve` and the `reject` functions. These are the only things that this function needs, nothing more nothing less.

So how about borrowing these functions from the `asyncController`?

As the `resolve` and the `reject` functions are defined privately inside the `asyncController`, we need something to throw inside the `asyncController` (send something as its argument), that the `resolve` and the `reject` functions can stick to it (I'm so sorry, this explanation is awful, let’s try to write the code to see what is in our mind).

```javascript
// ======================================
//   Inside async-controller.js module
// ======================================

function asyncController(borrowFunction) {
  var asyncHanlder = {};
  var _onSuccess = null;
  var _onError = null;

  try {
    borrowFunction(resolve, reject);
  } catch (err) {
    reject(err);
  }

  function resolve(data) {
    _onSuccess(data);
  }
  function reject(err) {
    _onError(err);
  }

  asyncHanlder.then = function (onSuccess, onError) {
    _onSuccess = onSuccess;
    _onError = onError;
  }

  return asyncHanlder;
}

module.exports = asyncController;
```

We defined a parameter named `borrowFunction`, which its job is to borrow the `resolve` and the `reject` functions. When the `asyncController` be run, the `borrowFunction` be run too, and also the `resolve` and the `reject` functions would be sent to it as its arguments.

Because the `borrowFunction` runs synchronously and the definition of this function is out of reach, we should somehow handle its possible errors, and the easiest way to do that is to put it inside a `try/catch` block (for now you can ignore the other parts).

```javascript
// ==========================
//   Inside fetch.js module
// ==========================

var asyncController = require('./async-controller');

function borrowFunction(resolve, reject) {

  fetchData(function (data) {
    if(data) {
      resolve(data);
    } else {
      reject('something is wrong!!!');
    }
  });

}

var asyncFetchData = asyncController(borrowFunction);
module.exports = asyncFetchData;
```

As we separated them into two files, here we must import the `asyncController` from its module.

To avoid confusion, first, just look at the piece of code that we called `fetchData` function and send a callback to it. This piece is the same `asyncFunction` that was previously in the `asyncController` function (we just changed its name back to the original one).

Except that piece, just reminds the definition of the `borrowFunction` that we used to send it to the `asyncController` to borrow the `resolve` and the `reject` functions from it.

Also the `asyncFetchData` is the same as the `asyncHandler` object, there is no difference.

Still the `App.js` is untouched.

From now on, for development we don’t need to change anything inside of the `fetch.js` anymore and all the operations placed in the `async-controller.js`.
