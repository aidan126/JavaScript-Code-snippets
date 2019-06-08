# reverseString_performance_test

```js
//output 
reverse 'how are you?' 1000000 times of each function

Built-in Methods: 669.812ms

forloop: 233.420ms

forof: 306.914ms

Foreach: 305.913ms

reduce: 278.284ms

Recursion: 519.227ms
```



 



```js
let str = 'how are you?';
var iterations = 1000000;
function reverseStr_BuiltinMethods(str) {
  return str
    .split('')
    .reverse()
    .join('');
}


function reverseStrforloop(str) {
  let revstr = '';
  for (let i = str.length - 1; i >= 0; i--) {
    revstr = revstr + str[i];
  }
  return revstr;
}

function reverseStrforof(str) {
  let revstr = '';
  for (let char of str) {
    revstr = char + revstr;
  }
  return revstr;
}

function reverseStringForeach(str) {
  let revSrring = '';
  str.split('').forEach(function(char) {
    revSrring = char + revSrring;
  });
  return revSrring;
}

function reverseStringReduce(str) {
  return str.split('').reduce(function(revString, char) {
    return char + revString;
  }, '');
}

function reverseStringRecursion(str) {
  if (str.length === 0) return '';

  return reverseStringRecursion(str.slice(1)) + str.slice(0, 1);
}

// -------------------------------

console.info(`reverse '${str}' ${iterations} times of each function`);

console.time('Built-in Methods');
for (let i = 0; i < iterations; i++) {
  reverseStr_BuiltinMethods(str);
}
console.timeEnd('Built-in Methods');

console.time('forloop');
for (let i = 0; i < iterations; i++) {
  reverseStrforloop(str);
}
console.timeEnd('forloop');

console.time('forof');
for (let i = 0; i < iterations; i++) {
  reverseStrforof(str);
}
console.timeEnd('forof');

console.time('Foreach');
for (let i = 0; i < iterations; i++) {
  reverseStringForeach(str);
}
console.timeEnd('Foreach');

console.time('reduce');
for (let i = 0; i < iterations; i++) {
  reverseStringReduce(str);
}
console.timeEnd('reduce');

console.time('Recursion');
for (let i = 0; i < iterations; i++) {
  reverseStringRecursion(str);
}
console.timeEnd('Recursion');

```

