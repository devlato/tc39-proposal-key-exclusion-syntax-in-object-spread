# Key Exclusion Syntax in Object Spread

## ECMAScript Proposal: Exclusion Syntax in Object Spread

**Author:** Denis Tokarev (Canva)  
**Champion:** TBD  
**Stage:** 0  

---

## Motivation

Since its introduction, the [object spread syntax](https://github.com/tc39/proposal-object-rest-spread) has become a cornerstone of modern JavaScript. Its declarative nature makes it highly readable and maintainable. However, current syntax falls short when developers need to exclude keys from objects, especially in complex use cases with dynamic keys, strings with spaces, or symbols.

---

## Proposal

### Key Exclusion Syntax: `-key`

The key exclusion syntax introduces a way to declaratively remove keys from objects during object spread operations. It works with:
- **Static keys**
- **Dynamic keys (e.g., values stored in variables)**
- **Complex key expressions**
- **Strings with spaces**
- **Symbols**

---

## Examples

### 1. Excluding Static Keys
The most basic use case, excluding specific keys directly in the spread syntax:

```js
const sanitizedOpts = {
  ...a,
  -key1,
  ...b,
  -key2,
};
```

### 2. Using Exclusion in the Middle of a Spread
Key exclusions can also be applied in the middle of a spread, affecting only the properties accumulated so far:

```js
const sanitizedOpts = {
  ...src,
  ...a,
  -key1, // Removes 'key1' from { ...src, ...a }
  ...b,
};
```

### 3. Excluding Keys with Spaces
Keys containing spaces can be excluded using either of these syntaxes:

```js
const sanitizedOpts = {
  ...a,
  -"key with space", // Excludes the key "key with space"
  -["another key with space"], // Alternate syntax for excluding "another key with space"
  ...b,
};
```

### 4. Excluding Dynamic Keys
Dynamic keys, such as those stored in variables, can also be excluded:

```js
const sanitizedOpts = {
  ...a,
  -[dynamicKey], // Excludes the value stored in `dynamicKey`
  ...b,
};
```

### 5. Using Symbols as Keys
Symbols can also be excluded:

```js
const sanitizedOpts = {
  ...a,
  -[Symbol.for("key")], // Excludes the Symbol `Symbol.for("key")`
  ...b,
};
```

### 6. Using Complex Expressions as Keys
Key expressions can be computed on-the-fly:

```js
const sanitizedOpts = {
  ...a,
  -[dynamicPrefix + "Id"], // Removes a computed key like "userId" if `dynamicPrefix` is "user"
  ...b,
};
```

---

## Behavior

### Execution Order
Key exclusions operate sequentially during object spreading. The syntax removes the specified keys from the **accumulated result** up to that point.

---

## Desugaring Examples

#### 1. Excluding Static Keys
Input:
```js
const sanitizedOpts = {
  ...src,
  -key1,
  ...a,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  for (const key in src) _$1[key] = src[key];
  delete _$1.key1; // Remove from the accumulated object
  for (const key in a) _$1[key] = a[key];
  return _$1;
})();
```

#### 2. Using Exclusion in the Middle of a Spread
Input:
```js
const sanitizedOpts = {
  ...src,
  ...a,
  -key1,
  ...b,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  for (const key in src) _$1[key] = src[key];
  for (const key in a) _$1[key] = a[key];
  delete _$1.key1; // Remove from the accumulated object
  for (const key in b) _$1[key] = b[key];
  return _$1;
})();
```

#### 3. Excluding Keys with Spaces
Input:
```js
const sanitizedOpts = {
  ...src,
  -"key with space",
  ...a,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  for (const key in src) _$1[key] = src[key];
  delete _$1["key with space"]; // Remove key with space
  for (const key in a) _$1[key] = a[key];
  return _$1;
})();
```

#### 4. Excluding Symbols
Input:
```js
const sanitizedOpts = {
  ...src,
  -[Symbol.for("key")],
  ...a,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  const excludedSymbol = Symbol.for("key");
  for (const key in src) _$1[key] = src[key];
  delete _$1[excludedSymbol]; // Remove symbol key
  for (const key in a) _$1[key] = a[key];
  return _$1;
})();
```

#### 5. Using Complex Expressions
Input:
```js
const sanitizedOpts = {
  ...src,
  -[dynamicPrefix + "Id"],
  ...a,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  const computedKey = dynamicPrefix + "Id";
  for (const key in src) _$1[key] = src[key];
  delete _$1[computedKey]; // Remove computed key
  for (const key in a) _$1[key] = a[key];
  return _$1;
})();
```

---

## Advantages

- **Supports Dynamic and Complex Keys**: Handles variables, strings with spaces, symbols, and expressions as keys.
- **Improved Readability**: Keeps code clean and declarative.
- **Potential Performance Gains**: May avoid unnecessary property deletions by skipping excluded keys during the copy process when feasible. In theory, this could ensure predictable Big O performance.
- **Reduced Boilerplate**: Simplifies exclusion patterns in complex merges.
- **Smaller Bundles**: Reduces the amount of code required to perform the key exclusion.

---

## Conclusion

The proposed key exclusion syntax enhances the flexibility and clarity of object spread operations. Its ability to handle dynamic keys, complex expressions, strings with spaces, and symbols makes it a robust tool for JavaScript developers. By eliminating unnecessary object copies or deletions, it also provides performance benefits in large-scale applications.

Contributions and feedback are welcome!

## Graveyard

### Excluding Multiple Keys Dynamically
You can exclude an array of keys using the spread operator within the exclusion syntax:

```js
const sanitizedOpts = {
  ...a,
  -[...keysToExclude], // Excludes all keys in the `keysToExclude` array
  ...b,
};
```

Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  for (const key in src) _$1[key] = src[key];
  for (const key of keysToExclude) delete _$1[key]; // Remove dynamic keys
  for (const key in a) _$1[key] = a[key];
  return _$1;
})();
```

## Discussion

* [(ES Discourse) Proposal: Key Exclusion Syntax in Object Spread](https://es.discourse.group/t/proposal-key-exclusion-syntax-in-object-spread/2238/2)

## TODO

The following is a high-level list of tasks to progress through each stage of the [TC39 proposal process](https://tc39.github.io/process-document/):

### Stage 1 Entrance Criteria

* [ ] Identified a "champion" who will advance the addition.
* [x] Prose outlining the problem or need and the general shape of a solution.
* [x] Illustrative examples of usage.
* [ ] ~~High-level API.~~

### Stage 2 Entrance Criteria

* [ ] [Initialspecification text.
* [ ] Transpiler support (_Optional_).

### Stage 2.7 Entrance Criteria

* [ ] Complete specification text.
* [ ] Designated reviewers have signed off on the current spec text:
  * [ ] Reviewer #1 has signed off
  * [ ] Reviewer #2 has signed off
* [ ] The ECMAScript editor has signed off on the current spec text.

### Stage 3 Entrance Criteria

* [ ] [Test262](https://github.com/tc39/test262) acceptance tests have been written for mainline usage scenarios and merged.

### Stage 4 Entrance Criteria

* [ ] Two compatible implementations which pass the acceptance tests: [1], [2].
* [ ] A pull request has been sent to tc39/ecma262 with the integrated spec text.
* [ ] The ECMAScript editor has signed off on the pull request.
