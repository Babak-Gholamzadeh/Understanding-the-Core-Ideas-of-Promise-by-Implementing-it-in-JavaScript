# Eliminate nesting by chainable object

So far, if we want to run several steps after resolving an async function, we have to put all of them inside a `then` method which makes it kind of messy. How about using `then` method repeatedly to run each step separately and respectively?

Let me show you the code to get the idea.

```javascript
// =================
//   Inside App.js
// =================

var asyncFetchData = require('./fetch.js');

asyncFetchData
  .then(function (result) {
    console.log(result); // output> data is here,
    return result + ' hooray!';
  })
  .then(function (result) {
    console.log(result); // output> data is here, hooray!
    return reverse(result);
  })
  .then(function (result) {
    console.log(result); // output> !yarooh ,ereh si atad
    throw 'something is wrong!';
    return result + ' hooray!';
  })
  .then(
    function (result) {
      console.log(result);
    },
    function (err) {
      console.log(err); // output> something is wrong!
    },
  );
```

Doesn’t it look nicer? Following up the steps and even handle the errors in this way seems much easier.

But don’t run this code; our program is not yet capable of doing that.

As now we exactly know what we desire, let’s think about the implementation of something like this.

I guess, before continuing, we should completely understand the concept of **chainable** objects. Don’t be scared, it’s much easier than it sounds.

## Create a chainable object

*Note: Feel free to skip this part if you already feel comfortable with that.*

Let’s start with an example. Imagine that we want to have a function that does some math stuff (just some simple operations), and whenever we call the function, it takes a single operand and gives us an object to handle the further operation (sometime we should make a long story short just by stop talking and showing the code).

```javascript
var operation = mathOperation(5);

var addedResult = operation.plus(2);
var subtractedResult = operation.minus(2);

console.log(addedResult); // output> 7
console.log(subtractedResult); // output> 3
```

Here the `operation` object contains the number 5 and it offers some methods that can operate over that number and return a new number.

Implementing the `mathOperation` function it’s kind of easy.

```javascript
function mathOperation(baseValue) {
  var operationHanlder = {};

  operationHanlder.plus = function(anotherValue) {
    return baseValue + anotherValue;
  };

  operationHanlder.minus = function(anotherValue) {
    return baseValue - anotherValue;
  };

  return operationHanlder;
}
```

At the first line the function just gets a parameter (i.e. that number 5) and then returns an object which has two methods (`plus` and `minus`).

And thanks to closure, we can access to the `baseValue` variable even after the function is executed.

So far so good. But what if we want to do several operations after each other? We don’t want to store the result after every single operation in a variable and call the `mathOperation` again. Come on, we are too lazy!

We want to do something like `operation.minus(4).plus(2).plus(3)` and so on.

Let’s take a closer look. Each period (`.`) operator offers the returned value of its left side operand.

Let’s zoom into it.

When we say `mathOperation(5).plus(2)`, the period operator saves the returned value of `mathOperation(5)` temporary in the memory and then executes its right operand base on that returned value which is stored before.

```javascript
var temp = mathOperation(5);
// Now 'temp' is an object which has a 'plus' method 
//   temp = {
//     plus: [Function]
//   }
var result = temp.plus(2);
```

But if you want to be able to do the same thing on the `result`, this variable must be an object that has a `plus` method. In other words, the `plus` method must return an object which has another `plus` method exactly like itself.

Of course we don’t want to repeat ourselves. We already have a function that creates an object with a `plus` method. Yeah! You guessed right, it’s the `mathOperation`. Why not we just use it again?

```javascript
operationHanlder.plus = function(anotherValue) {
  // return baseValue + anotherValue;
  return mathOperation(baseValue + anotherValue);
};
```

Great! Now because the `plus` method returns an object that has another `plus` method, we can call it repeatedly like this `mathOperation(5).plus(2).plus(3).plus(7)` and it works. We should also do the same thing for the `minus` method.

This action is called **`chaining`** because somehow it looks like a chain, and each method is connected to the previous one by period. Also the returned value of these methods because they have some methods that can be used to repeat this action again and again, they are called **`chainable`**.

But this action caused another problem for us. If all methods return an object that only has some other methods, how can we access our result value? It’s kept secretly inside the `mathOperation` function.

For solving this issue we can define another method that unlike the others, just simply returns our result value. Just a simple number not a fancy object.

```javascript
operationHanlder.getResult = function() {
  return baseValue;
};
```

Now is the time to gather all of them in one place and upgrade our `mathOperation` function.

```javascript
function mathOperation(baseValue) {
  var operationHanlder = {};

  operationHanlder.plus = function(anotherValue) {
    // return baseValue + anotherValue;
    return mathOperation(baseValue + anotherValue);
  };

  operationHanlder.minus = function(anotherValue) {
    // return baseValue - anotherValue;
    return mathOperation(baseValue - anotherValue);
  };

  operationHanlder.getResult = function() {
    return baseValue;
  };

  return operationHanlder;
}
```

And we can use it in this way:

```javascript
var operation = mathOperation(5);

// var addedResult = operation.plus(2);
// var subtractedResult = operation.minus(2);

var result_1 = operation
  .plus(2)
  .minus(4)
  .getResult();

var result_2 = operation
  .minus(4)
  .plus(2)
  .plus(3)
  .getResult();

console.log(result_1); // output> 3
console.log(result_2); // output> 6
```

At the end, when our operations are completed, we should call the `getResult` to get a simple number.

But, in my opinion, let’s go a little further with this `mathOperation` function before returning to the main topic and talk about asynchronous stuff. Because understanding this part could really help us to continue with more confidence.

If you have noticed, now the `mathOperation` are just able to do some a few operations. If some day another programmer wants to use this function for doing some other operations, must he ask us to add another method to the object? Why don’t we let him create his own custom operators and somehow tells the `mathOperation` to what to do?

## Create a thenable object

For implementing this idea we need some mechanism (that is supposed to be familiar to you) to takes an operator (which is just a function) and returns a chainable object that does the same thing.

```javascript
var operation = mathOperation(5);

operation
  .operator(function mul(val) {
    return val * 4;
  })
  .operator(function div(val) {
    return val / 2;
  });
```

Now the `operation` object just needs to have one single method and lets others tell it to what to do.

The structure of this pattern enables you to access the value (that was kept secretly inside the `mathOperation`) inside the function that you pass to the `operator`.

We didn’t invent this pattern, it has already existed and by convention our `operator` method is named `then`.

OK! Let’s create the `then` method.

```javascript
operationHanlder.then = function (operator) {
  var result = operator(baseValue);
  return mathOperation(result);
};
```

That’s it. Is it simple, yeah? It just takes an operator function, sends the `baseValue` to that operator and store the result of that. After that it puts the result value inside the `mathOperation` to get a chainable object.

```javascript
function mathOperation(baseValue) {
  var operationHanlder = {};

  operationHanlder.then = function (operator) {
    var result = operator(baseValue);
    return mathOperation(result);
  };

  return operationHanlder;
}

var operation = mathOperation(5);

operation
  .then(function (val) {
    return val + 2;
  })
  .then(function (val) {
    return val - 4;
  })
  .then(function (val) {
    console.log(val); // output> 3
  });
```

Oh look at this, we even don’t have to implement `getResult` method anymore. Because now we can access the result value directly, awsome!

And now you also know the definition of a `thenable` object. Simply when an object has a `then` method, it’s called **`thenable`**.

OK! Now we are equipped and we can go back to the main path to make our `asyncController` chainable.

As we have learned, if we want to enable our `then` method to return an object that also has a `then` method which is exactly like itself, we can (and should) use the `asyncController` function again to create the desired object.

But the only part that might get a little complex is that the `asyncController` takes a function as the argument. Do you remember the `borrowFunction`? We must use that function here again.

To prevent complexity, we add small pieces of code to the structure step by step. Don’t worry we don’t intend to rush.

As a reminder, let’s look at the current structure of the `then` method.

```javascript
asyncHanlder.then = function (onSuccess, onError) {
  _onSuccess = onSuccess;
  _onError = onError;
  executeController();
}
```

Just three tasks:

- Register the `onSuccess` callback
- Register the `onError` callback
- Run the `executeController`

Let’s start by adding a return statement at the bottom of the `then` function that returns the `ascynController` with a `borrowFunction`.

```javascript
asyncHanlder.then = function (onSuccess, onError) {
  _onSuccess = onSuccess;
  _onError = onError;
  executeController();

  function borrowFunction(resolve, reject) {
    // The party is here!
  }

  return asyncController(borrowFunction);
}
```

Till here, we have a thenable object which is returned from the `then` method, but it does nothing yet. For making this action to be useful, we ought to put some codes inside the `borrowFunction`.

In order to define the `borrowFunction` properly, we must understand the responsibility of this function at this position precisely.

To clarify the situation, let’s review in our mind that what exactly is going to happen on the other side of the hedge.

In a normal situation, we use an `asyncHandler` object, which has a `then` method to register callbacks for receiving the resolved value. But what will happen if we want to return a value in the callbacks (let’s keep going on just with the `onSuccess` callback to simplify, and ignore the `onError` for a few moments. It doesn’t matter, they both act kind of the same way).

```javascript
var asyncFetchData = require('./fetch.js');

asyncFetchData
  .then(function (result) {
    console.log(result); // output> data is here,
    return 5;
  });
```

Where does the value `5` go? This callback is the `onSuccess` callback that we defined as a parameter in the `then` method. The `onSuccess` will be assigned to the `_onSuccess` private variable, and finally it will be run in the `executeController` and unfortunately the returned value of that will be thrown away.

We need to find a way to store the value somewhere which be able to retrieve it somehow.

We can’t access the returned value of the `_onSuccess` in the `executeController`. Try to implement this idea just makes the complexity out of control (but if you are curious, you can try it on your own).

Take a look at the definition of the `then` method again. We already return an `asyncController` which has a `borrowFunction`. Remember guys that whatever we pass to the `resolve` method in the `borrowFunction`, it would be accessible via calling the `then` method and sending a callback to it.

OK! But where is the bridge between them? How do we suppose to get the returned value of the `onSuccess` and send it to the `resolve` function?

We know that if any kind of function that we assign to the `_onSuccess` would be called and the resolved value would be passed to it as an argument. How about define a new function and assign to the `_onSuccess` that just runs the `onSuccess` callback.

```javascript
_onSuccess = function(result) {
  var returnedValue = onSuccess(result);
}
```

I know this action seems useless, but we can put this definition inside the `borrowFunction` because over there we have access to the `resolve` function. And now we can send the `returnedValue` to the `resolve` function.

```javascript
function borrowFunction(resolve, reject) {
  _onSuccess = function(result) {
    var returnedValue = onSuccess(result);
    resolve(returnedValue);
  }
}
```

Now it looks magical, yeah?

Before ending this section, we should write some codes to prevent and handle possible errors.

First of all, let’s check if the `onSuccess` is being assigned or not. Because if it’s not, we should send the `result` value directly to the `resolve` function.

```javascript
function borrowFunction(resolve, reject) {

  _onSuccess = function(result) {

    if(!onSuccess) {
      resolve(result);
    } else {
      var returnedValue = onSuccess(result);
      resolve(returnedValue);
    }

  }

}
```

After that, because running the `onSuccess` might throw an error and this function is a sync function, it’s better to put it inside a `try/catch` block.

```javascript
function borrowFunction(resolve, reject) {

  _onSuccess = function(result) {

    if(!onSuccess) {
      resolve(result);
    } else {

      try {
        var returnedValue = onSuccess(result);
        resolve(returnedValue);
      } catch(err) {
        reject(err);
      }

    }

  }

}
```

Look at here. We can use the `reject` function to handle errors as well.

Defining the `_onError` function is kind of similar. The only thing that you ought to know about it is when you return a value inside the `onError` callback, it acts just like an `onSuccuss` callback. Because its value is a normal value and it must be treated as a normal value.

```javascript
_onError = function (err) {

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

};
```

If there wasn’t assigned any callback to the `onError`, we pass the error to the next round to be caught somewhere. But when everything is normal we only need to call the `resolve` function and send the `returnedValue` to it.

```javascript
asyncHanlder.then = function (onSuccess, onError) {

  function borrowFunction(resolve, reject) {

    _onSuccess = function (result) {
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
    };

    _onError = function (err) {
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
    };

    executeController();
  }

  return asyncController(borrowFunction);
}
```

Finally, we add the `executeController` at the end of the `borrowFunction`.

I just put the parts of code that were changed here. It means the other parts are untouched.

Now you can run this chainable code that was shown before:

```javascript
// =================
//   Inside App.js
// =================

var asyncFetchData = require('./fetch.js');

asyncFetchData
  .then(function (result) {
    console.log(result); // output> data is here,
    return result + ' hooray!';
  })
  .then(function (result) {
    console.log(result); // output> data is here, hooray!
    return reverse(result);
  })
  .then(function (result) {
    console.log(result); // output> !yarooh ,ereh si atad
    throw 'something is wrong!';
    return result + ' hooray!';
  })
  .then(
    function (result) {
      console.log(result);
    },
    function (err) {
      console.log(err); // output> something is wrong!
    },
  );
```
