---
title: The gist of MobX
sidebar_label: The gist of MobX
hide_title: true
---

<script async type="text/javascript" src="//cdn.carbonads.com/carbon.js?serve=CEBD4KQ7&placement=mobxjsorg" id="_carbonads_js"></script>

# The gist of MobX

## Concepts

MobX distinguishes the following three concepts in your application:

1. State
2. Actions
3. Derivations

Let's take a closer look at these concepts below, or alternatively, in the [10 minute introduction to MobX and React](https://mobx.js.org/getting-started) where you can interactively dive deeper into these concepts step by step and build a simple Todo list app using [React](https://facebook.github.io/react/) around it.

### 1. Define state and make it observable

<<<<<<< HEAD
<span class='definition'>_State_</span> is the data that drives your application.
Usually there is <span class='definition'>_domain specific state_</span> like a list of todo items and there is <span class='definition'>_view state_</span> such as the currently selected element.
<span class='important'>State is like spreadsheets cells that hold a value</span>.

<span class='important'>Store state in any data structure you like; plain objects, array, classes, cyclic data structures, references, it doesn't matter for the workings of MobX</span>.
<span class='important'>Just make sure that all properties that you want to change over time are marked <span class='definition'>`observable`</span> so that MobX can track them</span>. Here is a simple example:
=======
_State_ is the data that drives your application.
Usually, there is _domain specific state_ like a list of todo items, and there is _view state_, such as the currently selected element.
State is like spreadsheet cells that hold a value.

Store state in any data structure you like: plain objects, arrays, classes, cyclic data structures or references. It doesn't matter for the workings of MobX.
Just make sure that all properties you want to change over time are marked as `observable` so MobX can track them.

Here is a simple example:
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

```javascript
import { makeObservable, observable, action } from "mobx"

class Todo {
    id = Math.random()
    title = ""
    finished = false

    constructor(title) {
        makeObservable(this, {
            title: observable,
            finished: observable,
            toggle: action
        })
        this.title = title
    }

    toggle() {
        this.finished = !this.finished
    }
}
```

<<<<<<< HEAD
(Tip: this example could be <span class='important'>shortened</span> by using [`makeAutoObservable`](../refguide/observable.md) instead, but by being explicit we can show the different concepts in detail)

<span class='important'>Using `observable` is like turning a property of an object into a spreadsheet cell</span>.
But unlike spreadsheets, these values can be not only primitive values, but also references, objects and arrays.
=======
**Hint**: this example can be shortened using [`makeAutoObservable`](../refguide/observable.md), but by being explicit we can showcase the different concepts in greater detail.

Using `observable` is like turning a property of an object into a spreadsheet cell.
But unlike spreadsheets, these values can not only be primitive values, but also references, objects and arrays.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

But what about `toggle`, which we marked as `action`?

### 2. Update state using actions

An <span class='definition'>_action_</span> is any piece of code that changes the _state_. User events, backend data pushes, scheduled events, etc.
An action is like a user that enters a new value into a spreadsheet cell.

<<<<<<< HEAD
In the `Todo` model you can see that we have a method `toggle` that changes the value of `finished`. `finished` is marked as `observable`. <span class='important'>MobX recommends that you mark any code that changes `observable`s as [`action`](../refguide/action.md)</span>, so that MobX can automatically apply <span class='definition'>transactions</span> for optimal performance.

Using actions helps you to structure your code and prevents from inadvertently changing state when you don't want to.
<span class='important'>Methods that modify state are called _actions_</span> in MobX terminology. In contrast to <span class='definition'>_views_</span> which <span class='important'>compute new information based on the current state</span>.
=======
In the `Todo` model above you can see that we have a `toggle` method that changes the value of `finished`. `finished` is marked as `observable`. It is recommended that you mark any piece of code that changes `observable`'s as an [`action`](../refguide/action.md). That way MobX can automatically apply transactions for effortless optimal performance.

Using actions helps you structure your code and prevents you from inadvertently changing state when you don't want to.
Methods that modify state are called _actions_ in MobX terminology. In contrast to _views_, which compute new information based on the current state.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f
Every method should serve at most one of those two goals.

### 3. Create derivations that automatically respond to state changes
<span class='comment' data-comment='TODO: This section needs better explanation'></span>
_Anything_ that can be derived from the _state_ without any further interaction is a <span class='definition'>derivation</span>.
Derivations exist in many forms:

-   The _user interface_
-   _Derived data_, such as the number of remaining `todos`
-   _Backend integrations_, e.g. sending changes to the server

MobX distinguishes between two kinds of derivations:

<<<<<<< HEAD
-   <span class='definition'>_Computed values_</span>. These are <span class='important'>values that can always be derived from the current observable state</span> using a <span class='definition'>pure function</span>.
-   <span class='definition'>_Reactions_</span>. Reactions are <span class='important'>side effects that need to happen automatically if the state changes</span>. These are needed as a <span class='definition'>bridge between imperative and reactive programming</span>. Or to make it more clear, <span class='important'>they are ultimately needed to achieve I/O</span>.

People starting with MobX tend to use reactions too often.
The golden rule is: <span class='important'>if you want to create a value based on the current state, use `computed`</span>.

#### 3.1. Model derived values using computed

You created a <span class='definition'>computed value</span> by defining a property using a JS getter function (`get`) and then marking it with `computed` with `makeObservable`.
=======
-   _Computed values_, which can always be derived from the current observable state using a pure function
-   _Reactions_, side effects that need to happen automatically when the state changes (bridge between imperative and reactive programming)

When starting with MobX, people tend to overuse reactions.
The golden rule is, always use `computed` if you want to create a value based on the current state.

#### 3.1. Model derived values using computed

To create a *computed* value define a property using a JS getter function `get` and then mark it as `computed` with `makeObservable`.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

```javascript
import { makeObservable, observable, computed } from "mobx"

class TodoList {
    todos = []
    get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length
    }
    constructor() {
        makeObservable(this, {
            todos: observable,
            unfinishedTodoCount: computed
        })
    }
}
```

<span class='important'>MobX will ensure that `unfinishedTodoCount` is updated automatically when a todo is added or when one of the `finished` properties is modified</span>.

<<<<<<< HEAD
<span class='important'>Computations like these resemble <span class='definition'>formulas in spreadsheet programs</span> like MS Excel. They update automatically, but only when required</span>. That is, if something is interested in their outcome.

#### 3.2. Model side effects using reactions

<span class='important'>For you as a user to be able to see a change in state or computed values on the screen a <span class='definition'>_reaction_</span> is needed that repaints part of the GUI</span>.

<span class='important'>Reactions are similar to computed values, but instead of producing information, reactions produces <span class='definition'>side effect</span> like printing to the console, making network requests, incrementally updating the React component tree to patch the DOM, etc</span>.
=======
These computations resemble formulas in spreadsheet programs like MS Excel. They update automatically, but only when required. That is, if something is interested in their outcome.

#### 3.2. Model side effects using reactions

For you as a user to be able to see a change in state or computed values on the screen a _reaction_ is needed that repaints a part of the GUI.

Reactions are similar to computed values, but instead of producing information, reactions produce side effects like printing to the console, making network requests, incrementally updating React component tree to patch the DOM, etc.

>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f
In short, reactions bridge the worlds of [reactive](https://en.wikipedia.org/wiki/Reactive_programming) and [imperative](https://en.wikipedia.org/wiki/Imperative_programming) programming.

By far the most used form of reactions are UI components.
<span class='important'>Note that it is possible to trigger side effects from both actions and reactions</span>.
Side effects that have a clear, explicit origin from which they can be triggered, such
as making a network request when submitting a form, should be triggered explicitly from the relevant event handler.

#### 3.3. Reactive React components
<<<<<<< HEAD
<span class='comment' data-comment='TODO: This section needs better explanation'></span>
If you are using React, you can turn your components into reactive components by wrapping them with the [`observer`](http://mobxjs.github.io/mobx/react/react-integration.html) function from the `mobx-react-lite` package.
=======

If you are using React, you can make your components reactive by wrapping them with the [`observer`](http://mobxjs.github.io/mobx/react/react-integration.html) function from the bindings package you've [chosen during installation](installation.md#installation). In this example, we're going to use the more lightweight `mobx-react-lite` package.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

```javascript
import * as React from "react"
import { render } from "react-dom"
import { observer } from "mobx-react-lite"

const TodoListView = observer(({ todoList }) => (
    <div>
        <ul>
            {todoList.todos.map(todo => (
                <TodoView todo={todo} key={todo.id} />
            ))}
        </ul>
        Tasks left: {todoList.unfinishedTodoCount}
    </div>
))

const TodoView = observer(({ todo }) => (
    <li>
        <input type="checkbox" checked={todo.finished} onClick={() => todo.toggle()} />
        {todo.title}
    </li>
))

const store = new TodoList([new Todo("Get Coffee"), new Todo("Write simpler code")])
render(<TodoListView todoList={store} />, document.getElementById("root"))
```

<span class='definition'>`observer` converts React components into derivations of the data they render</span>.
When using MobX there are no smart or dumb components.
<<<<<<< HEAD
<span class='definition'>All components render smartly but are defined in a dumb manner. MobX will simply make sure the components are always re-rendered whenever needed, and never more than that</span>.
=======
All components render smartly, but are defined in a dumb manner. MobX will simply make sure the components are always re-rendered whenever needed, and never more than that.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

So the `onClick` handler in the above example will force the proper `TodoView` component to render as it uses the `toggle` action, but will only cause the `TodoListView` component to render if the number of unfinished tasks has changed.
And if you would remove the `Tasks left` line (or put that into a separate component), the `TodoListView` component will no longer re-render when ticking a task.

To learn more about how React works with MobX, check out [React integration](../react/react-integration.md).

#### 3.4. Custom reactions

You will need them rarely, but they can be created using the [`autorun`](../refguide/autorun.md#autorun),
[`reaction`](../refguide/autorun.md#reaction) or [`when`](../refguide/autorun.md#when) functions to fit your specific situations.
For example, the following `autorun` prints a log message every time the amount of `unfinishedTodoCount` changes:

```javascript
// A function that automatically observes the state.
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})
```
<<<<<<< HEAD
<span class='comment' data-comment='TODO: This needs better explanation'></span>
Why does a new message get printed each time the `unfinishedTodoCount` is changed? The answer is this rule of thumb:
=======

Why does a new message get printed every time the `unfinishedTodoCount` is changed? The answer is this rule of thumb:
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

_MobX reacts to any existing observable property that is read during the execution of a tracked function._

To learn more about how MobX determines which observables need to be reacted to, check out [understanding what MobX reacts to](../best/what-does-mobx-react-to.md).

## Principles

MobX supports a <span class='definition'>uni-directional data flow</span> where _actions_ change the _state_, which in turn updates all affected _views_.

![Action, State, View](../assets/action-state-view.png)

<<<<<<< HEAD
<span class='definition'>All _Derivations_ are updated **automatically** and **atomically**</span> when the _state_ changes. As a result it is <span class='important'>never possible to observe intermediate values</span>.

<span class='definition'>All _Derivations_ are updated **synchronously** by default</span>. This means that, for example, <span class='important'>_actions_ can safely inspect a computed value directly after altering the _state_</span>.
=======
All _derivations_ are updated **automatically** and **atomically** when the _state_ changes. As a result, it is never possible to observe intermediate values.

All _derivations_ are updated **synchronously** by default. This means that, for example, _actions_ can safely inspect a computed value directly after altering the _state_.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

<span class='definition'>_Computed values_ are updated **lazily**</span>. Any computed value that is <span class='important'>not actively in use will not be updated until it is needed for a side effect (I/O)</span>.
<span class='important'>If a view is no longer in use it will be garbage collected automatically</span>.

<<<<<<< HEAD
<span class='definition'>All _Computed values_ should be **pure**</span>. They are not supposed to change _state_.
=======
All _computed values_ should be **pure**. They are not supposed to change _state_.
>>>>>>> ff246b33e3d90337e970fbd20d930754de11688f

To learn more about the background context, check out [the fundamental principles behind MobX](https://hackernoon.com/the-fundamental-principles-behind-mobx-7a725f71f3e8).

## Try it out!

You can play with the above examples yourself on [CodeSandbox](https://codesandbox.io/s/concepts-principles-il8lt?file=/src/index.js:1161-1252).

## Linting

If you find it hard to adopt the mental model of MobX, configure it to be very strict and warn at runtime whenever you deviate from these patterns. Check out [linting MobX](../refguide/configure.md#linting-options).
