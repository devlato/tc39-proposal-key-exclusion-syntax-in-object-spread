# proposal-plus-minus-spread

Minus and plus spread operators proposal for JavaScript / ECMAScript


## Motivation

Since the object spread operator spec had landed, it has become widely used by devs. The reason for that is quite obvious – it's declarative, easily readable and straightforward way to produce an object from other objects. However, there are still use cases when devs have to write imperative code for performing quite simple operations. An example of such operation is filtering out an item from an object by key.

Let's illustrate this wth code. Let's assume we've got an object `obj`:

```js
const obj = {
  a: 1,
  b: {
    b1: 21,
    b2: 22,
  },
  c: 3,
};
```

If we want to take out an item from `obj.b` by its key (eg. `b1`) using a declarative paradigm, we have to write something like that (let's write it as a function, to cover a broader case when it does something meaningful before returning a result):

```js
const removeB1 = (src) => ({
  ...src,
  b: Object.keys(src.b).reduce((acc, key) => key === 'b1' ? acc : { ...acc, [key]: src.b[key] }, {}),
});
```

This line, in particular, requires some time to understand what's going on:

```js
  b: Object.keys(src.b).reduce((acc, key) => key === 'b1' ? acc : { ...acc, [key]: src.b[key] }, {}),
````


## Proposal: the minus operator

What I propose is to improve readability of that quite typical operation, by introducing a special operator, expressed by a minus sign `-`, into the language standard. This operator would allow omitting a key by prependng it with this operator, in the object spread body:

```js
const removeB1 = (src) => ({
  ...src,
  b: {
    ...src.b,
    -b1, // A minus operator!
  },
});
```

I propose using minus (`-`) sign because it doesn't conflict with existing language semantics, and thus is easier to implement.


### Execution order

Even though the minus operator is used in the object spread body, between the curly brackets, it must be applied to the result or spread operator.

Let's take an example:

```js
const result = {
  ...objA,
  ...objB,
  -keyToRemove,
};
```

It must be executed in the following order:
1. We merge objects `objA` and `objB` into a new object
2. We remove the key `keyToRemove` from that new object
3. We assign the result to a const `result`.

According to that, there's no difference at what position in the spread operator the minus operator is applied, so all these pieces of code would do the same:

```js
// Same
const result = {
  ...objA,
  ...objB,
  -keyToRemove,
};

// Also same!
const result = {
  ...objA,
  -keyToRemove,
  ...objB,
};

// And this!
const result = {
  -keyToRemove,
  ...objA,
  ...objB,
};
```

The second consequence is that in a particular object spread, a minus operator can be applied to each key only once.


### Alternatives

For sure, there are existing alternatives to the proposed minus operator.

First of all, it's the [delete operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete). Although, it cannot be used declaratively, meaning that the object produced by object spread must be assgned to a variable first, and only then we can remove the property from it by its key.

Second approach is using `Object.keys(obj).reduce(...)`, which is a block of repeated code that takes a bit of time to read and understand.

The third approach is even more imperative – it's `for ... of` loop or any related alternative.

The fourth possible solution is to manually assign all the properties, manually omitting the ones that has to be omitted. It's quite a lot boilerplate code.

And the last possible solution is to assign undefined to the key to be removed – but it's far from the ideal way to solve that issue, because the key would still exist in the object, meaning that if we enumerate through all the properties, there will be an undefined value for that key.


## Additional proposal: plus operator (optional)

For the API consistency, I'm proposing to add a plus operator (expressed by a prepending `+` sign), which would allow to forcefully specify a value for a given key in the same manner. The main benefit of that is ability to explicitely override a particular key, without having to care about code lines order.

```js
const result = {
  ...objA,
  ...objB,
  +keyToAdd: valueToAdd,
};
```

For instance:

```js
// Same
const result = {
  ...objA,
  ...objB,
  +keyToAdd: 'test',
};

// Also same!
const result = {
  ...objA,
  +keyToAdd: 'test',
  ...objB,
};

// And this!
const result = {
  +keyToAdd: 'test',
  ...objA,
  ...objB,
};
```

All the variants of the code from the example above must be executed in the following order:
1. We merge objects `objA` and `objB` into a new object
2. We assign a value `'test'` to a key `keyToAdd` to that new object
3. We assign the result to a const `result`.

As a consequence of that, we can assert that in a particular object spread, a plus operator can be applied to each key only once.
