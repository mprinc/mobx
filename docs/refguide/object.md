---
title: Observable Objects
sidebar_label: objects
hide_title: true
---

<script async type="text/javascript" src="//cdn.carbonads.com/carbon.js?serve=CEBD4KQ7&placement=mobxjsorg" id="_carbonads_js"></script>

# Observable Objects

Usage:

-   `observable.object(props, annotations?, options?)`

If a plain JavaScript object is passed to `observable` all properties inside will be <span class='important'>copied into a clone and made observable</span>.
(A <span class='definition'>plain object</span> is an object that wasn't created using a constructor function / but has `Object` as its prototype, or no prototype at all.)
`observable` is by default applied <span class='important'>recursively</span>, so if one of the encountered values is an object or array, that value will be passed through `observable` as well.

The `annotations` param can be used to override the declaration that is used for a specific property, like [`makeObservable` and `makeAutoObservable`](observable.md). See also [modifiers](modifiers.md).

```javascript
import { observable, autorun, action } from "mobx"

var person = observable(
    {
        // observable properties:
        name: "John",
        age: 42,
        showAge: false,

        // computed property:
        get labelText() {
            return this.showAge ? `${this.name} (age: ${this.age})` : this.name
        },

        setAge(age) {
            this.age = age
        }
    },
    {
        setAge: action
    }
)

// object properties don't expose an 'observe' method,
// but don't worry, 'mobx.autorun' is even more powerful
autorun(() => console.log(person.labelText))

person.name = "Dave"
// prints: 'Dave'

person.setAge(21)
// etc
```

Some things to keep in mind when making objects observable:

-   <span class='important'>Only plain objects will be made observable</span>. For <span class='definition'>non-plain objects</span> it is considered the <span class='important'>responsibility of the constructor to initialize the observable properties using [`makeObservable` or `makeAutoObservable`](observable.md)</span>.
-   <span class='definition'>Property getters</span> will be automatically turned into derived properties, just like declaring it [`computed`](computed) would do.
-   `observable` is applied recursively to a whole object graph automatically. <span class='important'>Both on instantiation and to any new values that will be assigned to observable properties in the future</span>. Observable will not recurse into non-plain objects.
-   These defaults are fine in 95% of the cases, but for more fine-grained on how and which properties should be made observable, see the [modifiers](modifiers.md) section.
-   Pass `{ deep: false }` as 3rd argument to disable the auto conversion of property values.
-   Pass `{ name: "my object" }` to assign a <span class='definition'>friendly debug name</span> to this object.

## `isObservableObject`

Usage:

-   `isObservableObject(value)`

Returns `true` if `value` is an observable object.

## Limitations in environments without Proxy support

When passing objects through `observable`, <span class='important'>only the properties that exist at the time of making the object observable will be observable</span>. Properties that are added to the object at a later time won't become observable, unless [`set`](object-api.md) or [`extendObservable`](extend-observable.md) is used. See also [limitations without proxies](../best/limitations-without-proxies.md)
