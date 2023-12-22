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

#### Why the dash/minus character?

The assumption is that it would be easier for the JS engines and transpilers to implement (because currently, the dash character is not expected before the key names in the object spread and thus won't conflict with any existing valid syntax).


#### Execution order

The key exclusion syntax would only be allowed at the end of the spread operator, before the closing curly bracket. Since the exclusion syntax removes the specified keys from the result, allowing it at other places inside the object spread (e.g., between multiple spread objects), is likely to cause an ambiguity.

```js
// When the key name is known statically
const sanitizedOpts = (opts) => {
  return {
    ...PRIVATE_OPTS,
    -keyThatMustNotBeThere, // SyntaxError: the key exclusion syntax can only be used at the end of the object spread
    ..opts,
  };
};
```


### The key inclusion syntax


## Specification

WIP


## Implementations

* Babel plugin – WIP
