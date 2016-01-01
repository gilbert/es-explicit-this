# ECMAScript Proposal: Syntax for Explicitly Naming `this`

> `this` isn't really a keyword, it is the natural "main" function argument in JavaScript.
> - (@spion)[https://github.com/mindeavor/es-pipeline-operator/issues/2#issuecomment-162348536]

This proposal extends the `function` declaration syntax to allow for explicit naming of what is normally called `this`. Its primary use case is to play nicely with the [function bind proposal](https://github.com/zenparsing/es-function-bind), making such functions more readable. For example:

```js
function array.zip (otherArray) {
  return array.map( (a, i) => [a, otherArray[i]] )
}

function subject.flatten () {
  return subject.reduce( (a,b) => a.concat(b) )
}

[10,20]
  ::zip( [1,2] )
  ::flatten()
//=> [10, 1, 20, 2]
```

## Motivation

The "keyword" `this` has been one of the greatest sources of confusion for programmers learning and using JavaScript. Fundamentally, it boils down to **two primary reasons**:

1. `this` is an **implicit** parameter, and
2. `this` **implicitly performs variable shadowing** in your functions.

Variable shadowing is true source of why `this` is so confusing to learn. In fact, variable shadowing is a bad practice in general. For example:

```js
function process (obj, name) {
  obj.taskName = name;
  doAsync(function (obj, amount) {
    obj.x += amount;
  });
};
```

In the above code, `obj.x` is referring to a different `obj` than `obj.name`. An experienced programmer would likely think this code is silly. Why name the inner parameter `obj` when there is already another variable in the same scope with the same name?

Yet, this behavior is exactly what `this` proceeds to do. If we were to translate the above example to use method-style functions:

```js
function process (name) {
  this.taskName = name;
  doAsync(function (amount) {
    this.x += amount;
  });
};
```

...we would end up with code *just as bad* as the original example. The second `this` is referring to a different object than the first `this`, even though they have the same name.

However, if we could **explicitly name** the object within the function parameters, we could disambiguate the two to avoid such variable shadowing, and make the above example look something more like this:

```js
function obj.process (name) {
  obj.taskName = name;
  doAsync(function result.callback (amount) {
    result.amount += 2;
  });
};
```

## Behavior

If a function explicitly names `this`, attempting to use `this` inside the body will throw an error:

```js
function elem.method (e) {
  console.log("Elem:", elem) // OK
  console.log("this:", this) // <-- Throws an error!
}
```

This behavior fits well with arrow functions, since they don't contain their own `this` in the first place.

```js
function elem.callback (e) {
  alert(`You entered: ${elem.value}`);
  setTimeout( () => elem.value = '', 1000 ) // OK
  setTimeout( () => this.value = '', 1000 ) // <-- Throws an error!
}
```

As normal, an inner `function` get its own `this`, which you can still choose whether or not to rename:

```js
runTask(function elem.cb (e) {

  elem.runAsyncTask(function () {
    console.log("Outer elem:", elem) // OK
    console.log("Inner this:", this) // OK
  });
})
```
