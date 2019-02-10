# lodash/fp <!-- omit in toc -->

[lodash](https://lodash.com/) is a library offering a set of JS utilities.

[lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide) is the functional programming version of its library.
The difference in this version is that most functions are curried and have the order of their arguments changes.

- [Object utilities](#object-utilities)
  - [getOr](#getor)
  - [pick](#pick)
  - [omit](#omit)
- [Array utilities](#array-utilities)
  - [difference](#difference)
  - [intersection](#intersection)
  - [without](#without)
  - [uniq](#uniq)
  - [sortBy](#sortby)
  - [`maxBy` and `minBy`](#maxby-and-minby)
- [Function utilities](#function-utilities)
  - [compose](#compose)
  - [flow](#flow)

Part of [Functional Programming for React](./README.md) series.

## Object utilities

### getOr

Gets the value at the `path` of the `object`. If it fails, `defaultValue` is returned.

The utility has multiple currying signatures:

```js
getOr(defaultValue, path, object);
getOr(defaultValue, path)(object);
getOr(defaultValue)(path)(object);
```

Example:

```js
const state = { deep: { list: [1, 2, 3] } };

const getListFrom = getOr([], ["deep", " list"]);

getListFrom(state); // [1, 2, 3]
getListFrom({}); // []
```

Note that the `getOr` utility is only included in `lodash/fp`.
The difference between `getOr` and `get` is that the latter does not receive a default value.

### pick

Returns an object containing only the picked properties.

Signatures:

```js
pick(paths, object);
pick(paths)(object);
```

Example:

```js
const pickFruits = pick(["apples", "oranges"]);
pickFruits({ apples: 3, candy: 2, oranges: 5 }); // { apples: 3, oranges: 5 }
```

### omit

The opposite of `pick`.
Returns an object containing all inital properties except the omitted ones.

Signatures:

```js
omit(paths, object);
omit(paths)(object);
```

Example:

```js
const omitFruits = pick(["apples", "oranges"]);
omitFruits({ apples: 3, candy: 2, oranges: 5 }); // { candy: 2 }
```

## Array utilities

### difference

Returns the items of the first array not found in the second.

Signatures:

```js
difference(array1, array2);
difference(array1)(array2);
```

Example:

```js
difference([1, 2, 3], [1, 2]); // [3]
```

Note: this function has unchanged argument order. The second argument is substracted from the first.

### intersection

Returns the items included in all given arrays.

Signatures:

```js
intersection(array1, array2);
intersection(array1)(array2);
```

Example:

```js
intersection([1, 2, 3], [3, 4]); // [3]
```

### without

Returns the items of the array, without the specified items.

Signatures:

```js
without(value, array);
without(value)(array);
```

Example:

```js
const withoutTwo = without(2);
withoutTwo([1, 2, 3]); // [1, 3]
```

### uniq

Returns unique items of an array.

```js
uniq([1, 2, 2, 3]); // [1, 2, 3]
```

### sortBy

Returns the sorted by the specified property.

Signatures:

```js
sortBy(property, array);
sortBy(property)(array);
```

Example:

```js
const sortByQuantity = sortBy("quantity");
sortByQuantity([
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "cherries", quantity: 0 },
]);
// [
//   { name: 'cherries', quantity: 0 },
//   { name: 'apples', quantity: 3 },
//   { name: 'oranges', quantity: 5 },
// ]

const sortByQuantityReverse = sortBy(({ quantity }) => -quantity);
// [
//   { name: 'oranges', quantity: 5 },
//   { name: 'apples', quantity: 3 },
//   { name: 'cherries', quantity: 0 },
// ]
```

### `maxBy` and `minBy`

Returns the maximum or minimum item of an array. The value of an item is determined either by picking a property or applying a function to each item.

Signatures

```js
maxBy(property, array);
maxBy(property)(array);
```

Example:

```js
const maxByQuantity = maxBy("quantity");
maxByQuantity([
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "cherries", quantity: 0 },
]); // { name: 'oranges', quantity: 5 }
```

## Function utilities

### compose

It composes functions in mathematical order.

Consider the following example:

```js
const sum = (a, b) => a + b;
const square = x => x * x;
const isBig = x => x > 100;

// const isBigSquareSum = (a, b) => isBig(square(sum(a, b)))
const isBigSquareSum = compose(
  isBig,
  square,
  sum
);
isBigSquareSum(5, 6); // true
```

Either of the two functions will output the same.

Note that the argument functions are given in the same order as they would normally be written in mathematics, but will be invoked in reverse order (starting with `sum`).

`compose` algorithm:

```js
const compose = (...fnArgs) => {
  const [first, ...funcs] = fnArgs.reverse();
  return (...args) => funcs.reduce((res, fn) => fn(res), first(...args));
};
```

The function returned by `compose` will take any number of arguments and pass them to the last function (`sum`).
Its result is passed to the previous function(`square`), and so on.
The result of the first function (`isBig`) will be returned.

### flow

It composes functions in argument order order.

```js
const sum = (a, b) => a + b;
const square = x => x * x;
const isBig = x => x > 100;

// const isBigSquareSum = (a, b) => isBig(square(sum(a, b)))
const isBigSquareSum = flow(
  sum,
  square,
  isBig
);

isBigSquareSum(5, 6); // true
```

Either of the two functions will output the same.

Note that the argument functions are given in the same order as they will be invoked (starting with `sum`).

`flow` algorithm:

```js
const flow = (first, ...funcs) => (...args) =>
  funcs.reduce((res, fn) => fn(res), first(...args));
```
