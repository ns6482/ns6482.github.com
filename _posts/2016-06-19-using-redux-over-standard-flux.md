---
layout: post
title: Experiment and journey with Redux
description: Case for using redux, sharing my learning experience and why to use it
modified: 2016-06-19
tags: [redux, aurelia, flux architecture]
comments: true
---

# Introduction

I wanted to go through my learning experience with Redux and hopefully in some way aid in understanding the core concepts around it. 

For me, it was a struggle initially, in particular it's difference around other flux implementations. What's the big deal with this and why all the buzz? Once I went through the key concepts, you suddenly realise it has many great things to offer.

To start with, I would like to focus on *why* redux more than anything.  I hope it helps you get it, and it just "clicks" and helps realise the power of using it.

For a detailed look I would strongly recommend the [excellent redux learning material by creator Dan Abramov](https://egghead.io/courses/getting-started-with-redux). I would also recommend this [excellent pluralsight video by Cody House](https://zombiecodekill.com/2016/05/22/react-and-redux-connecting-react-to-redux/).

Once we've covered the key concepts we can go through integration with our UI, with some basic code snippets. Finally, we will conclude by giving some examples when redux would be really useful in our application.

# Why redux

There are some great libraries out there to manage increasing complexity of front end apps.

Usually, if it's a straightforward simple app then vanilla JS does the job.

Some async, dom manipulation? Use jquery.

If our app demands more dom manipulation, async queries, routing then our all encompassing frameworks/micro-frameworks like React, Aurelia, Ember and Angular come to the rescue.

However, we still remain with some new problems - for e.g. managing state. It can get pretty messy when one area of the app updates a single state. So when we have lots of state updates and don't know exactly which process changed some state, or when. Or if there's a tonne of two-data binding, performance can suffer as well. So redux, is another flux library, it's probably worth starting by covering the key differences:

# Redux versus Flux

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/boltron/2670777828" title="Flux Capacitor"><img src="https://c5.staticflickr.com/4/3207/2670777828_1c698e648a_m.jpg" width="240" height="150" alt="Flux Capacitor"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

The thing you'll hear about redux is that *you'll know when to use it*. Hopfeully this statement will make sense by the end of this article.

Redux still follows the concept of uni-directional flow, same as flux.  
* You have an *action* that is dispatched
* A *store* (the store maintains state) that is updated
* The views subscribes to changes in the store. 

If an update to state is requested via the view, then it needs to occur through the same path, that is action, dispatcher, store. This means that data flows in one direction.

So in summary, we still have the idea of actions that define how state can be manipulated, and finally the concept of a store that holds state.

However, there are some key concepts you need to get your head around:

# You will not find these in other flux implementations...

## No dispatcher

The key difference in [flux is you have this singleton service, that dispatches actions](https://facebook.github.io/flux/docs/overview.html). The stores hook up to these dispatchers, and utilises event emitters to notify of changes to the store, views would typically subscribe to these events, update data from the store etc.

![Flux unidirectional flow](../images/flux.png)

However redux has no dispatcher, or singleton service that exists to submit actions. They use simple functions called reducers instead. For now, just know they are simple functions that update the state. We'll come back to them shortly. They are important!

![Redux flow](../images/redux.png)

## Single Store

In flux you can have *many stores with there corresponding states*. The idea with Redux is you have a **single store** that manages a single state. Advantages of having one state?

* simplification
* reliability
* debugging
* performance. 

Although there is one state, reducers effetively manage a 'slice' of our state or state domain. So we don't have to have this monolothic piece of code manageing one state. We have mentioned reducers a lot, let us move on to them next... 

## Reducers

Reducers. These are simple functions that always take in a state and action as parameters, so they follow this template...

```javascript
function addCommentReducer(state=[], action) {
  switch(action.type) {
    ...
    default: 
     return state;
  }
}
```

Few key elements to remember with reducer functions...

* you always pass in state and action object. 
* The action object always has a type property. 
* It's essential to return the default state.
* have a default state, using es6 ```=``` in the parameter.

Note that the switch statement is not imperative, you can still have if statements etc. as long as you return the default state.

The functions **always return a new state**. Reducers are 'pure functions' which means they have no side effects, don't call any other functions, or mutate the state in any way. So reducers look something like this:

```previous state + action => new state```

We can use various tools to make immutable objects, from es6's spread operator which you can use for arrays or  ```Object.assign``` and ```merge``` for objects.

It's usually good practice to keep our actions in a seperate file. For e.g. 

We might have  ```actions.js```, which contains an action that looks like this...

```javascript 
export const UPVOTE_COMMENT = 'UPVOTE_COMMENT';

export function upvoteCommentType(index) {
  return {
    type: UPVOTE_COMMENT, 
    index
  }
} 
```
Note that index property of ```index: index``` can be shortened to just ```index```. That's just a great es6 feature called *property shorthand notation*.

So our reducer, would look something like this.

```javascript
import {UPVOTE_COMMENT, upvoteComment} from './actions';

function comments(state = [], action) {
  if (action.type === UPVOTE_COMMENT) {
       return [
        ...state.slice(0, action.index),
          Object.assign({}, state[action.index], {voteCount: state.voteCount+1}),
        ...state.slice(action.index + 1)
      ];
  }
  return state
}
```
Note that, like in Abramov's video, it makes sense to break down our reducers, and *hunt* out for abstractions. For e.g. we could look at comments on a more granular level...

```javascript
const comment(state={count:0},action) {
  if(action.type === UPVOTE_COMMENT) {
   return Object.assign({},state, {voteCount: state.voteCount+1})
  }
  return state;
}

const comments(state = [], action) {
  if (action.type === UPVOTE_COMMENT) {
       return state.map(c => comment(c, action))
  }
  return state
}
```
So pass in an action that will make some change in state, our reducer will return a new state. Also important is the default, which returns the state unaltered. This is key because when we dispatch an action *every* reducer will be called. And, to reiterate, you must return a *new*, state. So let's discuss immutability...

## Immutability

Immutability can be a powerful practice, one key benefit is optimization. This is because it's quicker to compare values. If state was mutable, you would need to compare every single property. With immutable state? We just compare the reference. If they are different, we know a new state has been created. This is one huge way redux differs to other flux implementations. One thing is, this can be difficult to enforce on a team with many developers and so there are some libraries to enforce immutability.

## Combine reducers

Each reducer manages a 'slice of state' for your app. But actually, we mentioned there is one store that manages one state. So we need to combine all these reducer functions to return one big state object. That's where we can use the ```combineReducer``` function provided by Redux. This basically takes in your reducer functions, and from it, constructs a new state object, [see the docs for more on this](http://redux.js.org/docs/basics/Reducers.html#splitting-reducers
).
So in our example above, we could have the following reducers, combined to produce:

```javascript
const ourApp = (state = {}, action) {
  return {
    formWizard : formWizard(state.formWizard, action),
    session : session(state.session, action)
  }
}
```

Redux, provides a function that generates the above, we can just call:

```javascript
const ourApp = combineReducers(formWizard, sessionState)
```

You wouldn't typically reference state.sessionState, weird name right? Combine reducers iterates over your functions and maps a property to the state object, it will be referenced by name of the function you pass in. So for e.g. if we had...

```javascript
const ourApp = combineReducers(formWizard, sessionState)
```

our session would be referenced by ```state.sessionState.```. Since these function use ES6's property shorthand notation, we can do something like this to choose more meaningful names for our state...

```javascript
combineReducers({form: formWizard, session: sessionState})
```

[Dan Abramov's video illustrates how this function works really well](https://egghead.io/lessons/javascript-redux-reducer-composition-with-combinereducers)

# Connecting with our UI

The other key concept is getting all state management hooked to our UI. React-redux, or angular-redux, depending on your framework of choice is really great at decoupling presentational and container components.

I'm going to give an example using Aurelia as to how we can connect to redux, also refer to [this excellent todo app example](https://github.com/voidberg/aurelia-redux-todo/). They key is to break down our components and think of them in terms of presentational components and container components. So using the super simple comments example, our presentation component needs to be completly ignorant of redux...

## Creating dumb components, i.e. presentation first

Assuming we already have  a `<comment>` component, and we want to design a `<comments>` one as well, which will handle displaying comments,  the vote amount and the ability to upvote. Our first task would be to use the `bindable` decorater. 
The goal is to bind the comments state to the comments prop,  and the dispatcher function to upVoteCallback properties.

*comments.js*

```javascript
export class Comments {
  import {customElement, bindable} from 'aurelia-framework';
  
  @customElement('comments')

  export class Comments {
    @bindable comments;
    @bindable upvoteCallback;
  }
}
```

*comments.html*

```html
<template>
  <require from='./comment'></require>
    <ul class='comment-list'>
      <comment repeat.for='comment of comments' key='${$index}' comment.bind='comment'      click.delegate='$parent.upVoteCallBack($index)'>
      </comment>
    </ul>
</template>
```

So the above is completely dumb, and it's up to us to place these in a main entry point that does all the redux work and plug all the bindable properties, so for e.g. we could have...

*index.html*

```html
<template>
  <require from='./comments'></require>
  <div>
    <section class="comment">
		  <header id="header">
			  <h1>Comments</h1>
      </header>
      <comments comments.bind='comments'
        upvote-callback.call='upvote($event)'
      </comments>
    </section>
  </div>
</template>
```

*index.html*

```javascript
import { createStore } from 'redux';
import commentApp from './reducers';
...
}
```

## Container component -  subscribing to changes in the store

On our main entry point of the app, in this case, the container, we could have the following view model:

**index.js**

```javascript
constructor() {
  this.commentStore = createStore(commentApp);

  this.commentStore.subscribe(() =>
    this.update()
  );
  this.update();
}

update() {
  const state = this.commentStore.getState();
  this.comments = state.comments;
}
```

The above would ensure any changes dispatched to the store would render to the view. This is achieved via the ```update()``` callback function passed into ```subscribe```.

## Dispatching actions

Sending actions is pretty trivial, first import the action function...

``` javascript
import { upvoteComment} from './actions';
```

then proceed to update our state by dispatching actions, for e.g...

```javascript
upvote(index) {
  this.commentStore.dispatch(upvoteComment(index));
}
```

That nearly wraps it up for a very brief taster of redux.  We discussed the differences with the existing flux architectures. We went through the concepts of the store, reducers, single state. We talked about the importance of immutability and it being a key concept in redux. We also did a quick code run through of how to hook up to the UI. The latter having emphasis on separating the presentation and container components. 

I would like to finish by running through some examples that could warrant some redux usage:

# Redux use cases

Scenarios redux would welcome...

* **State updated in many ways** , for e.g. a state property could be updated by more than one area in the ui. Say an ecommerce website where you can increment your basket via a products page or checkout page. And we need to know which action was responsile for the update.

* **Good architecture**, decouple state logic completely from presentation layer. The ability to move state logic into another platform or server side.

* Want to be able to [undo, redo actions](https://github.com/omnidan/redux-undo). 

* **Excellent debugging and audit logging**, need to trace what a user exactly did. [This is possible without too much boiler plate](https://github.com/evgenyrodionov/redux-logger). 

* **Great Performance**, especially lots of ui updates with large data sets.

* **Maintenance**, break down complexity of business logic with improved testability due to pure functions and immutability.

* **Works with larger teams** - for e.g. developers can work independantly on reducers for specific state domains.

* **Need to persist the entire ui state** in the backend, or save snapshots, version of states that the UI needs to load from.

Hope you liked this article. If not, let me know. I'm looking to improve! In my next article I would like to run through developing a redux app step by step using Aurelia and Typescript. We will use some of the middleware like logging, go through the redux dev tools, and advanced topics like how to incorporate async calls using tools like Redux thunk.