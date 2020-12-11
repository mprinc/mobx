---
title: Update state using actions
sidebar_label: Actions
hide_title: true
---

<script async type="text/javascript" src="//cdn.carbonads.com/carbon.js?serve=CEBD4KQ7&placement=mobxjsorg" id="_carbonads_js"></script>

# Update state using actions

## Action

Usage:

-   `action` (annotation)
-   `action(fn)`
-   `action(name, fn)`

Any application has actions. <span class='important'>An <span class='definition'>action</span> is any code block that modifies state</span>.
<span class='important'>In principle actions always happen in response to an <span class='definition'>event</span></span>. For example a button was clicked, some input did change, a websocket message arrived, etc.
MobX requires that you declare your actions, though [makeAutoObservable](observable.md) can automate much of this job. Actions help you to structure your code better and offer performance benefits:

The `action` annotation should only be used on <span class='important'>functions that intend to _modify_ state.</span>
Functions that just perform look-ups, filter data, in short any function that <span class='definition'>derives information</span>, should _not_ be marked as actions; to allow MobX to track their invocations.

1. <span class='important'>Actions are run inside [transactions](api.md#transaction)</span>. <span class='important'>No observers will be updated until the outer-most action has finished</span>. This ensures that <span class='important'>intermediate or incomplete values produced during an action are not visible to the rest of the application</span> until the action has finished.
2. <span class='definition'>Outside actions it is (by default) not allowed to modify state</span>. This helps to <span class='important'>clearly identify in your code base where the state updates happen</span>.

### Examples

<!--DOCUSAURUS_CODE_TABS-->
<!--makeObservable-->

```javascript
import { makeObservable, observable, action } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeObservable(this, {
            value: observable,
            increment: action
        })
    }

    increment() {
        // intermediate states won't become visible to observers
        this.value++
        this.value++
    }
}
```

<!--makeAutoObservable-->

```javascript
import { makeAutoObservable } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeAutoObservable(this)
    }

    increment() {
        this.value++
        this.value++
    }
}
```

<!--action.bound-->

```javascript
import { makeObservable, observable, computed, action } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeObservable(this, {
            value: observable,
            increment: action.bound
        })
    }

    increment() {
        this.value++
        this.value++
    }
}

const doubler = new Doubler()
setInterval(doubler.increment, 1000) // calling increment this way is safe as it is bound already
```

<!--action(fn)-->
<span class='comment' data-comment='A good way to isolate actions out of the state object'></span>
```javascript
import { observable, action } from "mobx"

const state = observable({ value: 0 })

const increment = action(state => {
    state.value++
    state.value++
})

increment(state)
```

<!--runInAction(fn)-->

```javascript
import { observable } from "mobx"

const state = observable({ value: 0 })

runInAction(() => {
    state.value++
    state.value++
})
```

<!--END_DOCUSAURUS_CODE_TABS-->

### `action` can wrap functions

<span class='important'>To leverage the transactional nature of MobX as much as possible, actions should be passed as far outward as possible</span>. It is good to <span class='definition'>mark a class method as action</span> if it modifies state. It is even better to <span class='definition'>mark event handlers as action</span>; as it is the outer-most transaction that counts. <span class='important'>A single unmarked event handler that calls two actions subsequently would still generate two transactions</span>.

To help creating <span class='definition'>action based event handlers</span>, `action` is not only an annotation, but also a higher order function; it can be called with a function as argument and in that case it will return an `action` wrapped function with the same signature.

For example in React an `onClick` handler can be wrapped like:

```javascript
const ResetButton = ({ formState }) => (
    <button
        onClick={action(e => {
            formState.resetPendingUploads()
            formState.resetValues()
            e.stopPropagation()
        })}
    >
        Reset form
    </button>
)
```

For <span class='definition'>debugging purposes</span> we recommend either name the wrapped function, or pass a name as first argument to `action`.

<div class="detail">

<span class='important'>Another feature of actions is that they are [untracked](api.md#untracked)</span>; when an action is called from inside a side effect or computed value (a thing that should really rarily be needed!), observables read by the action won't be counted towards the dependencies of the derivation

`makeAutoObservable`, `extendObservable` and `observable` use a special flavour of `action`, <span class='important'><span class='definition'>`autoAction`</span>
that will determine at runtime if the function is a derivation or action</span>.

</div>

### `action.bound`

Usage:

-   `action.bound` (annotation)

The `action.bound` annotation can be used to <span class='important'>automatically bind a method to the correct instance</span>, so that `this` is always correctly bound inside the function.

<details id="avoid-bound"><summary>Tip: prefer arrow functions over `action.bound`<a href="#avoid-bound" class="tip-anchor"></a></summary>
If you want to bind actions in combination with `makeAutoObservable`, it is usually simpler to use arrow functions instead:

```javascript
import { makeAutoObservable } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeAutoObservable(this)
    }

    increment = () => {
        this.value++
        this.value++
    }
}
```

</details>

### `runInAction`

Usage:

-   `runInAction(fn)`

Use this utility to <span class='important'>create a temporarily action that is immediately invoked</span>. Can be <span class='important'>useful in asynchronous processes</span>.
See the [above code block](#examples) for an example.

## Asynchronous actions

In essence <span class='important'>asynchronous processes don't need any special treatment in MobX</span>, as <span class='important'>all reactions will update automatically regardless the moment in time they are caused</span>.
And <span class='important'>since observable objects are mutable, it is <span class='important'>generally safe</span> to <span class='definition'>keep references</span> to them during the duration of an action</span>.
However, <span class='important'>every step (tick) that updates observables in an asynchronous process <span class='comment' data-comment='Why? To keep transactions and avoid intermediate results or to optimize calculations? To me, it makes sense to make separate concepts as vuex has: mutations (for sync change) and actions (for async initiators)'>should be marked as `action`</span></span>.
This can be achieved in multiple ways by leveraging the above APIs, as shown below.

For example, when <span class='definition'>handling promises</span>, the handlers that update state should be wrapped using `action` or be actions, as shown below.

<!--DOCUSAURUS_CODE_TABS-->
<!--Wrap handlers in `action`-->

<span class='definition'>Promise resolution handlers</span> are handled in-line, but <span class='important'>run after the original action finished, so need to be wrapped by `action`</span>:

```javascript
import { action, makeAutoObservable } from "mobx"

class Store {
    githubProjects = []
    state = "pending" // "pending" / "done" / "error"

    constructor() {
        makeAutoObservable(this)
    }

    fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        fetchGithubProjectsSomehow().then(
            action("fetchSuccess", projects => {
                const filteredProjects = somePreprocessing(projects)
                this.githubProjects = filteredProjects
                this.state = "done"
            }),
            action("fetchError", error => {
                this.state = "error"
            })
        )
    }
}
```

<!--Handle updates in separate actions-->

If <span class='definition'>the promise handlers are class fields</span>, <span class='important'>they will automatically be wrapped in `action` by `makeAutoObservable`</span>:

```javascript
import { makeAutoObservable } from "mobx"

class Store {
    githubProjects = []
    state = "pending" // "pending" / "done" / "error"

    constructor() {
        makeAutoObservable(this)
    }

    fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        fetchGithubProjectsSomehow().then(
            this.projectsFetchSuccess,
            this.projectsFetchFailure
        )
    )

    projectsFetchSuccess = (projects) => {
        const filteredProjects = somePreprocessing(projects)
        this.githubProjects = filteredProjects
        this.state = "done"
    }

    projectsFetchFailure = (error) => {
        this.state = "error"
    }
}
```

<!--async/await + runInAction-->

<span class='definition'>Any steps after `await` aren't in the same tick</span>, so <span class='important'>need action wrapping</span>.
We can leverage `runInAction` here:

```javascript
import { runInAction, makeAutoObservable } from "mobx"

class Store {
    githubProjects = []
    state = "pending" // "pending" / "done" / "error"

    constructor() {
        makeAutoObservable(this)
    }

    fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = await fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            runInAction(() => {
                this.githubProjects = filteredProjects
                this.state = "done"
            })
        } catch (e) {
            runInAction(() => {
                this.state = "error"
            }
        }
    )
}
```

<!--`flow` + generator function -->

Flow is explained below.

```javascript
import { flow, makeAutoObservable, flowResult } from "mobx"

class Store {
    githubProjects = []
    state = "pending"

    constructor() {
        makeAutoObservable(this, {
            fetchProjects: flow
        })
    }

    // note the star, this a generator function!
    *fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = yield fetchGithubProjectsSomehow() // yield instead of await
            const filteredProjects = somePreprocessing(projects)
            this.state = "done"
            this.githubProjects = filteredProjects
        } catch (error) {
            this.state = "error"
        }
    }
}

const store = new Store()
const projects = await flowResult(store.fetchProjects())
```

<!--END_DOCUSAURUS_CODE_TABS-->

## 🚀 Using flow instead of async/await

Usage:

-   `flow` (annotation)
-   `flow(function* (args) { })`

<span class='important'>The <span class='definition'>`flow` wrapper</span> is an optional alternative to `async` / `await` that makes it easier to
work with MobX actions</span>.
`flow()` takes a [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) as its only input.
Inside the generator you can chain promises by yielding them (so instead of `await somePromise` you write `yield somePromise`).
The flow mechanism then will make sure the generator continues or throws when a yielded promise resolves.
So `flow` is an alternative to `async / await` that doesn't need any further `action` wrapping.
It can be applied as follows:

1. Wrap `flow` around your asynchronous function.
2. Instead of `async` you use `function *`.
3. Instead of `await` you use `yield`.

The [listing above](#examples) shows what this looks in practice.

<span class='important'>The `flowResult` function in the above listing is only needed when using TypeScript</span>.
Since decorating a method with `flow`, it <span class='important'>will wrap the returned generator in a promise</span>.
However, TypeScript isn't aware of that transformation, so, `flowResult` will make sure that TypeScript is aware of that type change.

`makeAutoObservable` and friends will automatically infer generators to be `flow`s.

<details id="flow-wrap"><summary>🚀 Using flow on object fields<a href="#flow-wrap" class="tip-anchor"></a></summary>
`flow`, like `action` can be used to wrap functions directly, so above example could also have been written as:

```typescript
import { flow, makeAutoObservable, flowResult } from "mobx"

class Store {
    githubProjects = []
    state = "pending"

    fetchProjects = flow(function* (this: Store) {
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = yield fetchGithubProjectsSomehow() // yield instead of await
            const filteredProjects = somePreprocessing(projects)
            this.state = "done"
            this.githubProjects = filteredProjects
        } catch (error) {
            this.state = "error"
        }
    })
}

const store = new Store()
const projects = await store.fetchProjects()
```

The upside is that we don't need `flowResult` anymore, the downside is that `this` needs to be typed to make sure its type is inferred correctly.

</details>

### 🚀 Flow cancellation

Another neat benefit of flows is that they are cancellable.
The return value of `flow` is a promise that resolves with the value that is returned from the generator function in the end.
The returned promise has an additional `cancel()` methods that will interrupt the running generator and cancel it.
Any `try / finally` clauses will still be run.

## 🚀 Disabling mandatory actions

By default, MobX 6 and later require that you use actions to make state changes.
You can however configure MobX to disable this behavior, see [`enforceActions`](configure.md#enforceactions).
This can be quite useful in for example unit test setup, where the warnings don't always have much value.
