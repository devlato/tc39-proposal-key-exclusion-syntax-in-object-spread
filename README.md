# Key exclusion and inclusion syntax in object spread

ECMAScript proposal and reference implementation for exclusion and inclusion syntax in the object spread.

**Author(s):** Denis Tokarev (Canva)

**Champion:** not identified

**Stage:** 0


## Motivation

Since its introduction to the specification, [the object spread syntax](https://github.com/tc39/proposal-object-rest-spread) has gained extreme popularity in the codebases of most organizations and open-source projects. Being declarative, the object spread syntax is easy to use, read, understand, and maintain.

However, when the use case is slightly more complex than just merging a few objects, the developers don't have the luxury of writing declarative code. 

Perhaps the most popular example is removing a key from the result object:

```js
// When the key name is known statically
const sanitizedOpts = (opts) => {
  const result = {
    ...PRIVATE_OPTS,
    ..opts,
  };

  // Removing the key "keyThatMustNotBeThere" from the result
  delete result.keyThatMustNotBeThere;

  return result;
};

// When there are multiple key names known statically
const sanitizedOpts = (opts) => {
  const result = {
    ...PRIVATE_OPTS,
    ..opts,
  };

  // Removing the key "keyThatMustNotBeThere" from the result
  delete result.keyThatMustNotBeThere;
  delete result.keyThatAlsoMustNotBeThere;

  return result;
};

// When the key name is not known beforehand
const sanitizedOpts = (opts) => {
  const result = {
    ...PRIVATE_OPTS,
    ..opts,
  };

  // Removing the key stored in KEY_THAT_MUST_NOT_BE_THERE from the result
  delete result[KEY_THAT_MUST_NOT_BE_THERE];

  return result;
};

// When there are multiple keys to remove
const sanitizedOpts = (opts) => {
  const result = {
    ...PRIVATE_OPTS,
    ..opts,
  };

  // Removing all the key names stored in KEYS_TO_REMOVE from the result
  KEYS_TO_REMOVE.forEach((key) => {
    delete result[key];
  });

  return result;
};
```

Removing keys this way has a few significant disadvantages:
- It is wordy.
- It is non-declarative and breaks the declarative paradigm of object spread.
- It makes the JS engine do extra work. First, the object spread will copy all the properties from all objects, and then we have to manually remove some of them, consuming additional CPU cycles, allocating the memory, and potentially, making the garbage collector care about a few more objects.

What if there was a way to give developers more declarative superpowers here?


## Proposed solution

### The key exclusion syntax

So, what if we could tell the JS engine not to copy some of the keys to the spread result at all? I am glad to present to you the key exclusion syntax, also mentioned as the minus syntax below.

Looking into the aforementioned examples, all the problems would be solved elegantly:

```js
// When the key name is known statically
const sanitizedOpts = (opts) => {
  return {
    ...PRIVATE_OPTS,
    ..opts,
    -keyThatMustNotBeThere, // The key exclusion syntax in action!
  };
};

// When there are multiple keys with known names
const sanitizedOpts = (opts) => {
  return {
    ...PRIVATE_OPTS,
    ..opts,
    -keyThatMustNotBeThere,
    -keyThatAlsoMustNotBeThere, // ...supporting multiple keys!
  };
};

// When the key name is not known beforehand
const sanitizedOpts = (opts) => {
  return {
    ...PRIVATE_OPTS,
    ..opts,
    -[KEY_THAT_MUST_NOT_BE_THERE], // ... and dynamic keys!
  };
};

// When there are multiple keys to remove
const sanitizedOpts = (opts) => {
  return {
    ...PRIVATE_OPTS,
    ..opts,
    -[...KEYS_TO_REMOVE], // ... and multiple dynamic keys!
  };
};
```

#### Why the dash/the minus character?

The assumption that it would be easier for the JS engines and transpilers to implement (because currently, the dash character is not expected before the key names in the object spread and thus won't conflict with any existing valid syntax).


### Execution order

Even though the minus operator is used in the object spread body, between the curly brackets, it must be applied to the result or spread operator (meaning that the  result would be order-independent; please see [this comment](https://github.com/devlato/proposal-plus-minus-spread/issues/3#issuecomment-740308156) for details).

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
