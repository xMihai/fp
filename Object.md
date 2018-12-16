# Review of Object methods <!-- omit in toc -->

- [Object.keys()](#objectkeys)
- [Object.values()](#objectvalues)
- [Object.entries()](#objectentries)

Part of [Functional Programming with ReactJS](./README.md) series.

## Object.keys()

Returns an array whose elements are strings corresponding to the enumerable properties found directly upon object.

```js
const bag = { apples: 3, bananas: 5, cherries: 0 };
const fruits = Object.keys(bag); // ["apples", "bananas", "cherries"]
```

## Object.values()

Returns an array whose elements are the enumerable property values found on the object.

```js
const bag = { apples: 3, bananas: 5, cherries: 0 };
const quantities = Object.values(bag); // [3, 5, 0]
const total = quantities.reduce((r, x) => r + x); // 8
```

## Object.entries()

Returns an array whose elements are arrays corresponding to the enumerable property `[key, value]` pairs found directly upon object.

```js
const bag = { apples: 3, bananas: 5, cherries: 0 };
const pairs = Object.entries(bag); // [['apples', 3], ['bananas', 5], ['cherries', 0]]
const strings = pairs.map(([name, quantity]) => `${name}: ${quantity}`); // ["apples: 3", "bananas: 5", "cherries: 0"]
```
