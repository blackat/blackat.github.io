---
layout: single
title: "Await/async pattern"
date: '2019-11-10'
last_modified_at: '2019-11-17'
comments: true
categories:
  - javascript, await/async
tags:
  - [javascript, async/await]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---

There are three main patterns to handle fetching external data for web applications: callbacks, promises, async/await.

## Callback

```javascript
function higherOrderFunction(s, callback) {
  return callback(s, 'world');
}

function concatenate(s1, s2) {
  return s1.concat(s2);
}

higherOrderFunction('hello', concatenate)
```

A `callback` is a generic function that is passed as an argument, a `higher order function` is a function that accept a callback as an argument.

A real world callback example `[1,2,3].map((i) => i + 10);`.

A callback can be used in both synchronous and asynchronous situations. The most common use in asynchronous case is when data as to be retrieved from a server, so *callback execution can be delayed until data are available*.

## Promise

Promises have been created to easily manage asynchronous requests. A promise have 3 states:

- pending, it is the initial state, neither fulfilled nor rejected;
- fulfilled, the operation has been successfully completed
- rejected, the operation failed.

We talk about *operation* since the promise pattern can be used in different fields: request/response, file reading etc.

What is a promise? A `Promise` is a *proxy* for a value not necessary known when the promise has been created. Handlers are associated to asynchronous success value or failure reason. *Asynchronous methods* return a promise object proxy instead of a value, the promise will return the value at some point.

```javascript
const promise = new Promise((resolve, reject){
  // asynchronous operation:
  setTimeout(() => {
    if(x > 1) {
      resolve(someValue); // change status to fulfilled
    } else {
      reject("sorry I have to reject"); // change status to rejected
    }
  }, 2000);
});
```

What has been created is just a JavaScript object, handlers can be attached via `Promise.prototype.then()` and `Promise.prototype.catch()` in case of success or rejection respectively.

```javascript
promise
.then(() => {
  console.log('😀');
})
.catch(() => {
  console.log('💩');
});
```

Promises can be chained, as seen in the example in a *sequential manner* improving the overall readability of the code. An old code based on callbacks can be replaces by promises improving the readability of the code, even if having many nested promises is not so simple.

```javascript
getUserToken('name', (token) => {
  getTweets(token, (tweets) => {
    searchText('text', (filteredTweets) => {
      updateComponent({tweets: filteredTweets})
    })
  }, callback(error))
}, callback(error));
```

Following code works thanks to the *implicit* return of the arrow function:

```javascript
getUserToken('name')
.then(getTweets(token)
  .then((tweets) => {
    searchText('text')
      .then(filteredTweets) => {
        updateComponent({tweets: filteredTweets})
      }))
.catch(function(error));
```

Reading the promise code notice that it is *sequential* so the *asynchronous* code looks like *synchronous*. The issue is that functions introduce new scopes, sibling functions can share previous results. Moreover the `.then()` introduces quite noise.

## async/await keywords

- adding `async` to a function implicitly return promises

```javascript
// asynchronous function declaration
async function readAsync() {
  return file;
}

// function return a promise
readAsync().then((file) => {
  console.log(file.name);
});
```

- `await` can only be used inside an `async` function, otherwise a syntax error will be thrown,

```javascript
async function readAsync() {
  await (file) => {
    console.log(file.name);
  }
}
```