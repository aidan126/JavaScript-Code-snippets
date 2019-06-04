

# take notes on promise



>  references 
>
> [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)




- ![1552564930881](C:\Users\soul-\drivehkaidan126\~2018秋\learn2018\typora-user-images\1552564930881.png)



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

- `return doSomethingElse()` will return a promise object 



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













