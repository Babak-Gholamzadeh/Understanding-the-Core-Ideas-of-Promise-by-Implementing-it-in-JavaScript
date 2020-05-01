# Start the journey from the simple callback pattern to promise

As we all know about callbacks and their restrictions, we should already know that if you want to access the result of an asynchronous function, you have to reach it inside of its callback. And of course this strategy ends to nested callbacks (aka callback hell).

The first step to avoid nesting callbacks is that we must find a way to access the async function’s result outside of its callback to be able to send the result around that we wouldn’t have to nest everything (the idea sounds a little foggy at this point, but stay with me).

Unfortunately, there is no magic way to do that and we must define something like a link in the ancestor scope of the callback and assign the resolved value of the async function to that link.

*Note: I use [CommonJS](http://wiki.commonjs.org/wiki/CommonJS) (i.e. [Node.js module ecosystem](https://nodejs.org/docs/latest/api/modules.html)) for modularize, but you can use any other types (e.g., [ES6 Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), [RequireJS](https://requirejs.org/), [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md), [UMD](https://github.com/umdjs/umd)) or create your own, It doesn’t matter.*
