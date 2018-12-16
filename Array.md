# Review of array methods <!-- omit in toc -->

- [.map()](#map)
- [.reduce()](#reduce)
- [.every()](#every)
- [.some()](#some)
- [.filter()](#filter)
- [.find()](#find)
- [.sort()](#sort)
- [.includes()](#includes)
- [.reverse()](#reverse)

Part of [Functional Programming with ReactJS](./README.md) series.

## .map()

Executes the provided function once for each element in an array, in order, and constructs a new array from the results.

It can be used to perform operations on each element:

```js
const list = [1, 2, 5];
const double = list.map(x => x * 2); // [2, 4, 10]
```

It can be used to extract information from lists of objects:

```js
const list = [
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "bananas", quantity: 0 },
];
const names = list.map(({ name }) => name); // ['apples', 'oranges', 'bananas']
```

## .reduce()

Executes the provided function once for each element in an array, in order, passing each result to the next call. It returns the last result.

```js
const list = [1, 3, 5];
const sum = list.reduce((result, x) => result + x, 0);
```

One common use if for indexing array items for fast access:

```js
const list = [
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "bananas", quantity: 0 },
];
const indexed = list.reduce(
  (result, item) => ({ ...result, [item.name]: item }),
  {}
);
// {
//   apples: { name: 'apples', quantity: 3 },
//   oranges: { name: 'oranges', quantity: 5 },
//   bananas: { name: 'bananas', quantity: 0 },
// }
```

## .every()

Tests whether all elements in the array pass the test implemented by the provided function.

The `every` method executes the provided function once for each element present in the array until it finds one where it returns a falsy value.
If such an element is found, the `every` method immediately returns `false`. Otherwise, it returns `true`.

```js
const list = [1, 2, 30, 40];
list.every(x => x < 10); // false
list.every(x => x < 100); // true
```

For empty arrays, the result is always `true`.

```js
const empty = [];
empty.every(x => false); // true
```

## .some()

Tests whether at least one element in the array passes the test implemented by the provided function.

The `some` method executes the provided function once for each element present in the array until it finds one where callback returns a truthy value.
If such an element is found the `some` method immediately returns `true`.
Otherwise, it returns `false`.

```js
const list = [1, 2, 30, 40];
list.some(x => x > 10); // true
list.some(x => x > 100); // false
```

For empty arrays, the result is always `false`.

```js
const empty = [];
empty.every(x => true); // false
```

## .filter()

Creates a new array with all elements that pass the test implemented by the provided function.

The `filter` executes the provided function once for each element in an array, and constructs a new array of all the values for which callback returns a truthy value.

```js
const list = [1, 2, 30, 40];
list.filter(x => x < 10); // [1, 2]
```

## .find()

Method introduced in ES6.

Returns the first element in the array that passes the test implemented by the provided function.

The `find` method executes the provided function once for each index of the array until it finds one where callback returns a truthy value. If such an element is found, `find` immediately returns that element. Otherwise, it returns `undefined`.

```js
const list = [
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "bananas", quantity: 0 },
];
list.find(({ name }) => name === "apples"); // { name: 'apples', quantity: 3 }
list.find(({ name }) => name === "cherries"); // undefined
```

Handling a default value to prevent errors:

```js
const list = [
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "bananas", quantity: 0 },
];
list.find(({ name }) => name === "apples").quantity; // 3
list.find(({ name }) => name === "cherries").quantity; // TypeError: Cannot read property 'quantity' of undefined
const q = (list.find(({ name }) => name === "cherries") || { quantity: 0 })
  .quantity; // 0
```

Before ES6:

```js
function find(array, testFn) {
  var i = 0;
  while (i < array.length) {
    if (testFn(array[i])) return array[i];
    i++;
  }
}
```

## .sort()

Sorts the elements of the array.
The method mutates the array and also returns it.

To prevent mutating the original array, clone it before reversing.

```js
const list = [1, 100, 2];
const sorted = [...list].sort(); // [1, 2, 100]
```

If a comparison function is not supplied, all non-`undefined` array elements are sorted by converting them to strings and comparing strings in UTF-16 code units order.
For example, "banana" comes before "cherry", but because numbers are converted to strings, "80" comes before "9" in Unicode order.
All `undefined` elements are placed at the end of the array.

If a comparison function is supplied, all non-`undefined` array elements are sorted according to the return value of the compare function.
All `undefined` elements are sorted to the end of the array, with no call to the comparison function.

The comparison function receives two items, `a` and `b`, and must return a number.
It must always return the same value when given a specific pair of elements `a` and `b`. If inconsistent results are returned then the sort order is undefined.

Returned value:

- if the comparison function returns 0, the order of `a` and `b` is not changed
- if the comparison function returns a number less than 0, `a` will come before `b` in the array
- if the comparison function returns a number greater than 0, `a` will come after `b` in the array

```js
const list = [
  { name: "apples", quantity: 3 },
  { name: "oranges", quantity: 5 },
  { name: "bananas", quantity: 0 },
];
const byQuantityDesc = (a, b) => b.quantity - a.quantity;
const sorted = [...list].sort(byQuantityDesc);
```

## .includes()

Method introduced in ES7.

Determines whether an array includes a certain element.
Strict equality is used for comparison.
Returns a `Boolean` which is `true` if the provided value is found within the array.

```js
const list = [1, 2, 3];
list.includes(2); // true
list.includes(4); // false
list.includes(true); // false
```

Before ES7:

```js
const list = [1, 2, 3];
const exists = list.indexOf(2) > -1; // true
```

## .reverse()

Reverses the order of the elements.
The method mutates the array and also returns it.

To prevent mutating the original array, clone it before reversing.

```js
const list = [1, 2, 3];
const reversed = [...list].reverse(); // [3, 2, 1]
```
