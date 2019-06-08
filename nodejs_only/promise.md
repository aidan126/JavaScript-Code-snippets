# take notes on promise



>  references 
>
>  [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)
>
>  [handling errors with async/await and promises](<https://medium.com/@vcarl/handling-errors-with-async-await-and-promises-cd2fea534f08>)
>
>  [Asynchronous JavaScript: From Callback Hell to Async and Await](https://medium.freecodecamp.org/avoiding-the-async-await-hell-c77a0fb71c4c)



![alt](https://storage.googleapis.com/converbucket/1552564930881.png)







- `resolve` and `reject` callbacks will be mapped to `Promise.then` and `Promise.catch`methods respectively
- One thing should be kept in mind: **once a Promise is created, its execution is already happening**, no matter if you `.then` it or not.

- why promise?
  - callbacks  deprive us of keywords like `return` and `throw`
  - deprive us of the *stack*



**Q: What is the difference between these four promises?**

```js
doSomething().then(function () {
  return doSomethingElse();
}).then(finalHandler);

doSomething().then(function () {
  doSomethingElse();
}).then(finalHandler);

doSomething().then(doSomethingElse())
  .then(finalHandler);

doSomething().then(doSomethingElse);
```

- firstly, to understand `then` format

  - `then(resolveHandler)`

  - `then(resolveHandler, rejectHandler)`

  - so, you should put **function declaration(函數聲明)after then**

- what can we do after `then`
  
  - `return` another promise
  - `return` a synchronous value (or `undefined`)
  - `throw` a synchronous error
  
    





```javascript
// Puzzle #1
doSomething().then(function () {
  return doSomethingElse();
}).then(finalHandler);

doSomething
|-----------------|
                  doSomethingElse(undefined)
                  |------------------|
                                     finalHandler(resultOfDoSomethingElse)
                                     |------------------|

```

- Notice that I'm `return`ing the second promise – that `return` is crucial. If I didn't say `return`, then the `getUserAccountById()` would actually be a *side effect*, and the next function would receive `undefined` instead of the `userAccount`.

> A side effect is any application state change that is observable outside the called function other than its return value. Side effects include:
>
> - Modifying any external variable or object property (e.g., a global variable, or a variable in the parent function scope chain)
> - Logging to the console
> - Writing to the screen
> - Writing to a file
> - Writing to the network
> - Triggering any external process
> - Calling any other functions with side-effects



```javascript
//Puzzle #2
doSomething().then(function () {
  doSomethingElse();
}).then(finalHandler);

doSomething
|-----------------|
                  doSomethingElse(undefined)
                  |------------------|
                  finalHandler(undefined)
                  |------------------|
    
```

- `doSomethingElse()` don't return a promise object , so 2nd `then` depends on `doSomething()`promise object



```javascript
//Puzzle #3
doSomething().then(doSomethingElse())
  .then(finalHandler);

doSomething
|-----------------|
doSomethingElse(undefined)
|---------------------------------|
                  finalHandler(resultOfDoSomething)
                  |------------------|
```

- `doSomethingElse()` is a function call instead of function declaration, so, it will execute 
- 2 and 3 are similar ,  both put a function call inside `then` 



```js
//Puzzle #4
doSomething().then(doSomethingElse)
  .then(finalHandler);

doSomething
|-----------------|
                  doSomethingElse(resultOfDoSomething)
                  |------------------|
                                     finalHandler(resultOfDoSomethingElse)
                                     |------------------|
```

- `doSomethingElse` is a function declaration, so it works , it is same with Puzzle #1

  



the demo of test above

[promiseTest]:(C:\Users\soul-\Desktop\temp2\save\promiseExplain.js)



```javascript
/**
 * ! this file demonstrate the use of Promise,
 * ! more info refer to learn2018\javascript\nodejs_only\promise.md
 */

/**
 * ! return 1 after 1sec
 */
function doSomething() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1);
    }, 1000);
  });
}

/**
 * ! return 2 after 2sec
 */
function doSomethingElse() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2);
    }, 2000);
  });
}

function finalHandler() {
  console.log(`elapsed time:`, (Date.now() - startTime) / 1000);
}

// ----  main --------
var startTime = Date.now();

/**
 * !Puzzle #1
 * !output: elapsed time: 3.019
*/
doSomething()
  .then(function() {
    return doSomethingElse();
  })
  .then(finalHandler);


/**
 * !Puzzle #2
 * !output: elapsed time: 1.012
*/ 
doSomething()
  .then(function() {
    doSomethingElse();
  })
  .then(finalHandler);


/**
 * !Puzzle #3
 * !output: elapsed time: 1.008
*/
doSomething()
  .then(
    doSomethingElse()
  )
  .then(finalHandler);


 /**
 * !Puzzle #4
 * !output: elapsed time: 3.019
 */
doSomething()
  .then(
    doSomethingElse
  )
  .then(finalHandler);

```



---



### Mistakes



#### #1 promise with `forEach` or `loop` and other iteration

```javascript
// I want to remove() all docs
db.allDocs({include_docs: true}).then(function (result) {
  result.rows.forEach(function (row) {
    db.remove(row.doc);  
  });
}).then(function () {
   // it isn't waiting on anything, and can execute when any number of docs have been removed! 
  // I naively believe all docs have been removed() now!
});


//the correct way
db.allDocs({include_docs: true}).then(function (result) {
  return Promise.all(result.rows.map(function (row) {
    return db.remove(row.doc);
  }));
}).then(function (arrayOfResults) {
  // All docs have really been removed() now!
});

```

- wrap all in `Promise.all()`
- `Promise.all()` return a Promise if every promise in array resolves

- `Promise.all()` also passes an array of results to the next function





#### #2 forgetting to add .catch()





#### #3 using side effects instead of returning

```javascript
somePromise().then(function () {
  someOtherPromise();
}).then(function () {
  // Gee, I hope someOtherPromise() has resolved!
  // Spoiler alert: it hasn't.
});

//the correct way already explain in #puzzle 1
```



#### #4 what if I want the result of two promises?

```js
getUserByName('nolan').then(function (user) {
  return getUserAccountById(user.id);
}).then(function (userAccount) {
  // dangit, I need the "user" object too!
});
```

- 1 is save the intermediate result in a global variable, but it is kludge 

  > A kludge or kluge is a workaround or quick-and-dirty solution that is clumsy, inelegant, inefficient, difficult to extend and hard to maintain
  >
  > workaround. 雖不能根本解決, 但能避開問題的替代方法

- 2

  - ```js
    getUserByName('nolan').then(function (user) {
      return getUserAccountById(user.id).then(function (userAccount) {
        // okay, I have both the "user" and the "userAccount"
      });
    });
    ```



#### #5: promises fall through

```js
Promise.resolve('foo').then(Promise.resolve('bar')).then(function (result) {
  console.log(result);
});
// it print out foo
```

- The reason this happens is because when you pass `then()` a non-function (such as a promise), it actually interprets it as `then(null)`, which causes the previous promise's result to fall through. 

- you *can* pass a promise directly into a `then()` method, but it won't do what you think it's doing. `then()` is supposed to take a function, so most likely you meant to do:

  -  ```js
    Promise.resolve('foo').then(function () {
      return Promise.resolve('bar');
    }).then(function (result) {
      console.log(result);
    });
    ```

- practice: always pass a function into `then()`!











### practice



#### change callback to promise 

```js
new Promise(function (resolve, reject) {
  fs.readFile('myfile.txt', function (err, file) {
    if (err) {
      return reject(err);
    }
    resolve(file);
  });
}).then(/* ... */)
```



#### always return or throw* from inside a `then()`function

- Unfortunately, there's the inconvenient fact that non-returning functions in JavaScript technically return `undefined`, which means it's easy to accidentally introduce side effects when you meant to return something.

  For this reason, I make it a personal habit to *always return or throw* from inside a `then()`function. I'd recommend you do the same.





#### `then(resolveHandler).catch(rejectHandler)`isn't exactly the same as `then(resolveHandler, rejectHandler)`

```js
//catch(), which is just sugar for then(null, ...)
                                      
//I said above that catch() is just sugar. So these two snippets are equivalent:

somePromise().catch(function (err) {
  // handle error
});

somePromise().then(null, function (err) {
  // handle error
});

//However, that doesn't mean that the following two snippets are equivalent:

somePromise().then(function () {
  return someOtherPromise();
}).catch(function (err) {
  // handle error
});

somePromise().then(function () {
  return someOtherPromise();
}, function (err) {
  // handle error
});


// consider this code 
somePromise().then(function () {
  throw new Error('oh noes');
}).catch(function (err) {
  // I caught your error! :)
});

somePromise().then(function () {
  throw new Error('oh noes');
}, function (err) {
  // I didn't catch your error! :(
});
```

- because As it turns out, when you use the `then(resolveHandler, rejectHandler)` format, the `rejectHandler` *won't actually catch an error* if it's thrown by the `resolveHandler` itself.
- a personal habit to never use the second argument to `then()`, and to always prefer `catch()`.







-------



### Handling errors with async/await and promises



#### Regular way

```javascript
async () => {
  try {
    const data = await fetchData();
    doSomethingComplex(data);
  } catch (e) {
    // Any errors thrown by `fetchData` or `doSomethingComplex` are caught.
  }
}

//  2 different way of handle Promise 
() => {
  fetchData()
    .then(data => {
      doSomethingComplex(data);
    })
    .catch(err => {
      // Errors from `fetchData` and `doSomethingComplex` end up here.
    });
};

fetchData()
    .then(
      data => {
        doSomethingComplex(data);
      },
      err => {
        // Only errors from `fetchData` are caught.
      }
    );
};
```



#### with .catch()

- error handling is relatively self-contained
- requires you to add a null check on your data

```js
async () => {
  const data = await fetchData().catch(e => {
    // Only errors from `fetchData` are caught.
  });
  if (!data) return;
  doSomethingComplex(data);
};
```

