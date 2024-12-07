# Key Exclusion Syntax in Object Spread

## ECMAScript Proposal: Exclusion Syntax in Object Spread

**Author:** Denis Tokarev (Canva)  
**Champion:** TBD  
**Stage:** 0  

---

## Motivation

Since its introduction, the [object spread syntax](https://github.com/tc39/proposal-object-rest-spread) has become a cornerstone of modern JavaScript. Its declarative nature makes it highly readable and maintainable. However, current syntax falls short when developers need to exclude keys from objects, especially in complex use cases with multiple spreads.

---

## Proposal

### Key Exclusion Syntax: `-key`

Introducing a concise, declarative way to exclude keys at any point in an object spread expression. The syntax dynamically removes the specified keys from all properties accumulated up to that point.

---

## Examples

### Basic Example: Key Exclusion at the End of Spread

The exclusion syntax can be used to remove keys from the final merged result:

```js
const sanitizedOpts = (opts) => ({
  ...src,
  ...a,
  ...b,
  -keyToExclude, // Removes 'keyToExclude' from the final merged result
});
```

### Using Exclusion in the Middle of a Spread

The exclusion syntax can also be used in the middle of a spread expression, removing keys from the object as it is being built:

```js
const sanitizedOpts = (opts) => ({
  ...src,
  ...a,
  -key1, // Removes 'key1' from { ...src, ...a }
  ...b,  // Spread 'b' after 'key1' is excluded
});
```

### Multiple Exclusions at Different Points

Keys can be excluded multiple times at different points in the spread expression:

```js
const sanitizedOpts = (opts) => ({
  ...src,
  -key1, // Removes 'key1' from { ...src }
  ...a,
  -key2, // Removes 'key2' from { ...src, ...a }
  ...b,
});
```

---

## Behavior

### Desugaring Examples

#### Exclude Key at the End
Input:
```js
const sanitizedOpts = (opts) => ({
  ...src,
  ...a,
  ...b,
  -keyToExclude,
});
```
Desugared:
```js
const sanitizedOpts = (opts) => {
  const _$1 = { ...src, ...a, ...b };
  delete _$1.keyToExclude;
  return _$1;
};
```

#### Exclude Key in the Middle
Input:
```js
const sanitizedOpts = (opts) => ({
  ...src,
  ...a,
  -key1,
  ...b,
});
```
Desugared:
```js
const sanitizedOpts = (opts) => {
  const _$1 = { ...src, ...a };
  delete _$1.key1;
  const _$2 = { ..._$1, ...b };
  return _$2;
};
```

#### Multiple Exclusions at Different Points
Input:
```js
const sanitizedOpts = (opts) => ({
  ...src,
  -key1,
  ...a,
  -key2,
  ...b,
});
```
Desugared:
```js
const sanitizedOpts = (opts) => {
  const _$1 = { ...src };
  delete _$1.key1;
  const _$2 = { ..._$1, ...a };
  delete _$2.key2;
  const _$3 = { ..._$2, ...b };
  return _$3;
};
```

---

### Performance-Optimized Desugaring Example

For maximum performance, excluded keys can be omitted during the object-building process entirely, avoiding unnecessary copying or deletion.

#### Exclude Key Without Any Copying
Input:
```js
const sanitizedOpts = (opts) => ({
  ...src,
  ...a,
  -key1,
  ...b,
});
```
Optimized Desugared Code:
```js
const sanitizedOpts = (opts) => {
  const _$1 = {};
  for (const key in src) {
    if (key !== 'key1') _$1[key] = src[key];
  }
  for (const key in a) {
    if (key !== 'key1') _$1[key] = a[key];
  }
  for (const key in b) {
    _$1[key] = b[key]; // No exclusions here since key1 has already been removed
  }
  return _$1;
};
```

#### Exclude Multiple Keys Without Any Copying
Input:
```js
const sanitizedOpts = (opts) => ({
  ...src,
  ...a,
  -[...keysToExclude],
  ...b,
});
```
Optimized Desugared Code:
```js
const sanitizedOpts = (opts) => {
  const _$1 = {};
  for (const key in src) {
    if (!keysToExclude.includes(key)) _$1[key] = src[key];
  }
  for (const key in a) {
    if (!keysToExclude.includes(key)) _$1[key] = a[key];
  }
  for (const key in b) {
    _$1[key] = b[key]; // No exclusions here since all keys in keysToExclude are skipped before this point
  }
  return _$1;
};
```

---

## Advantages

- **Improved Readability**: Keeps code clean and declarative.
- **Fine-Grained Control**: Allows exclusions at specific points in the spread expression.
- **Performance Gains**: Avoids unnecessary property deletions by not copying excluded keys.
- **Reduced Boilerplate**: Simplifies exclusion patterns in complex merges.

---

## Conclusion

The proposed key exclusion syntax improves the flexibility and clarity of object spread operations, addressing common pain points in JavaScript development. Its ability to operate contextually within spread expressions and its potential for performance optimization make it a valuable addition to the language.

Contributions and feedback are welcome!
