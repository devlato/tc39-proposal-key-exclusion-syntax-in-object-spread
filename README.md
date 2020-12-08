# proposal-plus-minus-spread

Minus and plus operators: a spread operator enhancement proposal for JavaScript / ECMAScript


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

1. The [delete operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete). Although, it cannot be used declaratively, meaning that the object produced by object spread must be assgned to a variable first, and only then we can remove the property from it by its key:
```js
const obj = {
  a: 'aValue',
  b: 'bValue',
  key: 'value',
};

// To avoid mutating the original object, we create a copy
const result = { ...obj };
// Then we imperatively delete the key
delete result.key;
// And since the above line isn't an rvalue, we have to return the result explicitly
return result;
```

2. `Object.keys(obj).filter(...).reduce(...)` which is a block of repeated code that takes a bit of time to read and understand:
```js
const obj = {
  a: 'aValue',
  b: 'bValue',
  key: 'value',
};

// A bit wordy, ineffective, and too difficult to read
return Object
    .keys(obj)
    .filter((k) => k !== 'key')
    .reduce((acc, k) => ({ ...acc, [k]: obj[k] }), {});
```

3. The third approach is even more imperative – it's `for ... of` loop or any related alternative.
```js
const obj = {
  a: 'aValue',
  b: 'bValue',
  key: 'value',
};

// A bit wordy `for ... of` loop
const result = {};
for (const [key, value] of Object.entries(obj)) {
   if (key !== 'key') {
      result[key] = value;
   }
}
// Also, loops aren't rvalues and thus, we have to return explicitly
return result;
```

4. Another possible solution is to manually assign all the properties, manually omitting the ones that has to be omitted. It's quite a lot boilerplate code.
```js
const obj = {
  a: 'aValue',
  b: 'bValue',
  key: 'value',
};

// `obj` might have too many properties here and this might be inconvenient
// to manually enumerate all of them
return {
  a: obj.a,
  b: obj.b,
};
```

5. Using spread with destructuring operator, to filter out all the undesired properties (for example, [`const { propertyIDontWant, ...newObject } = origObject; return newObject;`](https://github.com/devlato/proposal-plus-minus-spread/issues/1)).
```js
const obj = {
  a: 'aValue',
  b: 'bValue',
  key: 'value',
};

// `result` can't be returned directly from here, and also, an unnecessary variable `key` is created
const { key, ...result } = obj;
// So we have to return explicitly
return result;
```

6. And the last possible solution is to assign undefined to the key to be removed – but it's far from the ideal way to solve that issue, because the key would still exist in the object, meaning that if we enumerate through all the properties, there will be an undefined value for that key.
```js
const obj = {
  a: 'aValue',
  b: 'bValue',
  key: 'value',
};

return {  
  ...obj,
  // Even though `obj[key]` is `undefined`, `obj.hasOwnProperty('key')` would return true,
  // producing undesired side effects
  key: undefined,
};
```


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
