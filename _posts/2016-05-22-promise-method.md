---
layout: post
title: Dealing with synchronous and asynchronous using Promise.method
description: Using Promise.method and a brief glimpse into the world of async and await
modified: 2016-05-17
tags: [promises, javascript, bluebird, es7, async, await]
comments: true
---

### Promises are cool, but there's this thing...

Promises are great, and they are great in helping us solve the callback hell scenario many of us are familiar with. But we are still left with some potentially new problems, for e.g. when we have a combination of sync and async actions or errors to deal with. And we are left with the task of turning everything into a promise. It's very easy for this to add overhead to our code and cause readability to suffer to some extent.

### Introducing Promise.method

The aim of this post is to illustrate how we can use the Bluebird promise library to help tidy up code that requires both async and sync calls, one such feature is Promise.method. We'll then move on from Promise.method evolving to using es7 features, which simplifies everything using the async and await methods.

### Typical example

One example you may often come across is retrieving data from a cache, and should it not be in the cache it gets data from a database instead. So our method could make a sync or async call. Either way it has to return a promise. So say we have some database service with these functions:

```js
function getUserId(userId) {

  let user = dataCache.getUser(userId)); //sync call

  if (user) { // user in cache
    return Promise.resolve(user); //wrap value in promise
  } else //async call
    return asyncdbGetUser(userId).then(data) {  // returns promise
      user = data.user;
      dataCache.setUser(user);
      return user;
    };
}
```

In the above code snippet, the cache call is synchronous and the database call is asynchronous

One way we can simplify this, and aid readability is to use Bluebird's `Promise.method`

```js
function getUserId(userId) {
  Promise.method(() => { //wraps in Promise
    let user = dataCache.getUserId(userId));

    if (user) {
      return user; //no need to use Promise.resolve now
    } else
    return asyncdbGetUser(userId).then(data) {
      user = data.user
      dataCache.setUser(user);
      return user;
    };
  })

}
```

The result is the call is easier to read and takes out the effort of 'promisifying' the call. It doesn't interfere with our core logic in anyway. The previous example demanded we resolve the cached user in the promise. The use of `Promise.method` would be useful if we had many sync and async calls together.

### Handling sync errors

Bluebird also has a similar tool to convert synchronous errors to rejected promises, `Promise.try`. To my knowledge appears that Promise.method does the same thing. The reason you may use the latter is more for intent (for converting a sync error to a rejected promise) and easier readability. Correct me otherwise Bluebird team.

So we now know, that something along these lines could be handled by our neat little Promise.method.

```js
function getUserId(userId) {
  Promise.method(() => {

    if (!user) {
      throw new Error ('must enter user id');
    }
    ...

  })

}
```

### Using async and await ES7

Now the features discussed above are in Bluebird only, but a similar way you can 'wrap' values in promises are using `async` and `await`. Provided 'asyncdbGetUser' returns a promise we can use await. Either way, if it wasn't the case it would wrap it in one.

SYou can use the es7 future today usinga transpiler like Babel, see  https://github.com/babel/babel/tree/master/packages/babel-plugin-transform-async-functions. So, you can now do something like this:

```js
async function getUserId(userId) {

  if (!user) {
    throw new Error ('must enter user id');
  }

  let user = null;
  user = dataCache.getUserId(userId));

  if (user) {
     return user;
  } else
    user =  await asyncdbGetUser(userId).then(data);
    dataCache.setUser(user);
  }
}
```
One thing to be wary of, the above will swallow up our error. It's up the consumer to utilise a proper try and catch. For further reading on this, and using async await please go to https://pouchdb.com/2015/03/05/taming-the-async-beast-with-es7.html.

### Conclusion

Hope this short post has helped in some way illustrate how we can use Promise.method to help readability of our code and avoid extra overhead in Promisfying values, and although covered briefly, we can see async and await offers a cool alternative to Promise.method.
