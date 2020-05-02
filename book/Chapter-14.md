# Implement `resolve` and `reject` as static functions

Having some static functions can make our function handier.

Start with the `resolve` function. This function takes a value and returns a thenable object from the `asyncController`.

```javascript
asyncController.resolve = function (value) {

  function borrowResolve(resolve) {
    resolve(value);
  }

  return asyncController(borrowResolve);
}
```

It's simple and we resolve the value straight away.

The implementation of the `reject` function is also the same, but this time we use the `reject` parameter of the `borrowFunction`.

```javascript
asyncController.reject = function (err) {

  function borrowReject(resolve, reject) {
    reject(err);
  }

  return asyncController(borrowReject);
}
```

And to finish this tutorial, you can just change the name of the `asyncController` function to `Promise`.

<br>

<p align="center">
  <b>
    Congratulations. You have implemented and also understood the promise pattern.
  </b>
</p>
