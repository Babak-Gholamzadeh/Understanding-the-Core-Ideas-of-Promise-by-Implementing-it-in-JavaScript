# Implement `catch` method

I know our program is capable to handle the errors now, but adding a `catch` method next to the ‘then’ method causes our program to look more beautiful.

In order to implement this method you just need to know one thing.

Let me reveal the secret of the `cache` method here.

In fact, the `catch` method is the `then` method that its first parameter is `null`. That’s it.

```javascript
asyncHanlder.catch = function (onError) {
  return asyncHanlder.then(null, onError);
}
```

Add this code after the definition of the `then` method.

Definitely this section is the shortest.
