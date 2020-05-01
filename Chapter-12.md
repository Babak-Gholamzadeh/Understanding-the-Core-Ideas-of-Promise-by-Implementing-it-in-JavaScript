# Register several callbacks before resolving

What do you think will happen if we call the `then` method on the same object a few times before resolving?

Look at the example below to understand the situation better.

```javascript
var asyncFetchData = require('./fetch.js');

asyncFetchData
  .then(function (result) {
    console.log('result 1:', result); // output> result 1: data is here!
    return result + ' hooray!';
  })
  .then(function (result) {
    console.log(result); // output> data is here, hooray!
  });

asyncFetchData
  .then(function (result) {
    console.log('result 2:', result); // output> result 2: data is here!
  });
```

We didn’t even put the second one inside a `setTimeout` and both of the callbacks would be registered before resolving.

What would be in the output?

I’m telling you, when the `then` method on the second `asyncFetchData` be called, its callback will overwrite the previous ones and the output would be:

> output> result 2: data is here!

For solving this problem we can just register the callbacks inside an array. Also in the `executeController` we loop over the array and run all of them and then make the array empty, because each callback must be called once.

At first we can get rid of the `_onSuccess` and the `_onError` variables and instead of them define an array named `onThenHandlers` (I'm sorry again, this time for my lack of talent in choosing a better name).

In the `borrowFunction` inside the `then` method, we use the same code but instead of assign that functions to the `_onSuccess` and the `_onError` variables, we define them as the methods of a new object and finally push this object to the `onThenHandlers` array.

```javascript
function borrowFunction(resolve, reject) {

  var hanlder = {

    onSuccess: function (result) {
      if (!onSuccess) {
        resolve(result);
      } else {
        try {
          var returnedValue = onSuccess(result);
          resolve(returnedValue);
        } catch (err) {
          reject(err);
        }
      }
    },

    onError: function (err) {
      if (!onError) {
        reject(err);
      } else {
        try {
          var returnedValue = onError(err);
          resolve(returnedValue);
        } catch (err) {
          reject(err);
        }
      }
    }

  };

  onThenHanlers.push(hanlder);

  executeController();
}
```

Also for running the methods inside this array we have to tweak the `executeController`.

```javascript
function executeController() {

  if(state === 'PENDING') {
    return;
  }

  setTimeout(function () {
    // if (state === 'RESOLVED') {
    //     _onSuccess(value);
    // } else {
    //     _onError(value);
    // }

    onThenHanlers.forEach(function (hanlder) {

      if (state === 'RESOLVED') {
        hanlder.onSuccess(value);
      } else {
        hanlder.onError(value);
      }

    });
    onThenHanlers = [];

  }, 0);
  
}
```

We just loop over the array and run the corresponding method based on the state, then after the loop the array must become empty.

At the beginning of the `executeController`, we checked the state, because we didn’t want that some callbacks come here and make the array empty before resolving the value.

Because now the `then` method of the `asyncHandler` object literary can be run everywhere, at any time, and any number of times, it’s a good decision to make sure that the async functions be resolved once and their state and value be immutable.

To achieve that, we need to check the current state before changing anything in the `applyNewState` function.

```javascript
function applyNewState(newState) {
  return function (newValue) {

    if (isThenable(newValue)) {
      return newValue.then(resolve, reject);
    }

    if (state === 'PENDING') {
      value = newValue;
      state = newState;
      executeController();
    }

  }
}
```

Look at the code, the `state` and the `value` only can be changed if the current state is in the `PENDING` mode, even we avoid to run `executeController`.

Now it’s more reliable.
