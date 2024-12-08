# Key Exclusion Syntax in Object Spread

## ECMAScript Proposal: Exclusion Syntax in Object Spread

**Author:** Denis Tokarev (Canva)  
**Champion:** TBD  
**Stage:** 0  

---

## Motivation

Since its introduction, the [object spread syntax](https://github.com/tc39/proposal-object-rest-spread) has become a cornerstone of modern JavaScript. Its declarative nature makes it highly readable and maintainable. However, current syntax falls short when developers need to exclude keys from objects, especially in complex use cases with dynamic keys or intermediate exclusions.

---

## Proposal

### Key Exclusion Syntax: `-key`

The key exclusion syntax introduces a way to declaratively remove keys from objects during object spread operations. It works with:
- **Static keys**
- **Dynamic keys (e.g., values stored in variables)**
- **Complex key expressions**

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

### 2. Excluding Dynamic Keys (including Symbols)
Dynamic keys, such as those stored in variables, can also be excluded:

```js
const sanitizedOpts = {
  ...a,
  -[dynamicKey], // Excludes the value stored in `dynamicKey`
  ...b,
};
```

### 3. Excluding Multiple Keys Dynamically
You can exclude an array of keys using the spread operator within the exclusion syntax:

```js
const sanitizedOpts = {
  ...a,
  -[...keysToExclude], // Excludes all keys in the `keysToExclude` array
  ...b,
};
```

### 4. Using Complex Expressions as Keys
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
Key exclusions operate sequentially during object spreading. The syntax removes the specified keys from all properties accumulated **up to that point**.

```js
const sanitizedOpts = {
  ...src,
  ...a,
  -key1, // Removes 'key1' from { ...src, ...a }
  ...b,
};
```

### Desugaring Examples

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
  for (const key in src) {
    if (key !== 'key1') _$1[key] = src[key];
  }
  for (const key in a) {
    _$1[key] = a[key];
  }
  return _$1;
})();
```

#### 2. Excluding Dynamic Keys
Input:
```js
const sanitizedOpts = {
  ...src,
  -[dynamicKey],
  ...a,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  for (const key in src) {
    if (key !== dynamicKey) _$1[key] = src[key];
  }
  for (const key in a) {
    _$1[key] = a[key];
  }
  return _$1;
})();
```

#### 3. Excluding Multiple Dynamic Keys
Input:
```js
const sanitizedOpts = {
  ...src,
  -[...keysToExclude],
  ...a,
};
```
Desugared:
```js
const sanitizedOpts = (() => {
  const _$1 = {};
  for (const key in src) {
    if (!keysToExclude.includes(key)) _$1[key] = src[key];
  }
  for (const key in a) {
    _$1[key] = a[key];
  }
  return _$1;
})();
```

#### 4. Using Complex Expressions
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
  for (const key in src) {
    if (key !== computedKey) _$1[key] = src[key];
  }
  for (const key in a) {
    _$1[key] = a[key];
  }
  return _$1;
})();
```

---

## Advantages

- **Supports Dynamic and Complex Keys**: Handles variables and expressions as keys.
- **Improved Readability**: Keeps code clean and declarative.
- **Performance Gains**: Avoids unnecessary property deletions by not copying excluded keys.
- **Reduced Boilerplate**: Simplifies exclusion patterns in complex merges.

---

## Conclusion

The proposed key exclusion syntax enhances the flexibility and clarity of object spread operations. Its ability to operate contextually and handle complex, dynamic keys makes it a powerful tool for JavaScript developers. By eliminating unnecessary object copies or deletions, it also provides performance benefits in large-scale applications.

Contributions and feedback are welcome!
