# Functional programming <!-- omit in toc -->

> Functional programming is the process of building software by composing pure functions, avoiding shared state, mutable data, and side-effects.
> Functional programming is declarative rather than imperative, and application state flows through pure functions.
> Contrast with object oriented programming, where application state is usually shared and colocated with methods in objects.
> (Source: [Medium](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0))

- [First class functions](#first-class-functions)
  - [Higher-order functions](#higher-order-functions)
- [Pure functions](#pure-functions)
  - [Memoization](#memoization)
  - [Side-effects](#side-effects)
- [Immutability](#immutability)
  - [Native immutability in Javascript](#native-immutability-in-javascript)
  - [Immutability for React](#immutability-for-react)
- [Declarative programming](#declarative-programming)
  - [Javascript statements](#javascript-statements)
- [Currying](#currying)
- [Composition](#composition)

Part of [Functional Programming for React](./README.md) series.

## First class functions

In functional programming, functions are first class values.
Functions are treated like any other variable.
They can be assigned to variables or constants, sent as arguments to other functions or returned by other functions.

Javascript treats functions as first class values:

```js
// functions can be assigned to variables or constants
const sum = (a, b) => a + b;
sum(1, 2);

// functions can be sent as arguments
const calculate = (operation, a, b) => operation(a, b);
calculate(sum, 1, 2);

// functions can return a function
const getOperation = flag => (flag ? sum : diff);
calculate(getOperation(true), 1, 2);
```

### Higher-order functions

A **higher order function** is a function that receives a function as argument or returns a function.
In contrast, first order functions do not receive functions as argument nor return a function.

```js
const sum = (a, b) => a + b;
const hof = fn => (...args) => {
  const result = fn(...args);
  console.log(args, result);
  return result;
};
const loggingSum = hof(sum);
```

## Pure functions

Pure functions must be cacheable.
If their arguments were to be stored in a cache together with the function result, retrieving the result should be equivalent to invoking the function.

A pure function is one that respects the principles of predictability and self-containment.

A function is **predictable** if it always returns the same result when given the same parameters.
This means that no external input may be used in computing the result.

A function is **self-contained** if it does not create any side-effects. This means that no changes to the parameters or external variables may be made.
All effects must be included in the returned value.

```js
const sum = (a, b) => a + b;
```

Relying on any external data to compute the result makes a function _impure_:

```js
const factor = 5;
const multiply = list => list.map(x => x * factor);
multiply([1, 2, 3]); // [5, 10, 15]
```

But it can be rewritten as a pure function by including in its parameters all needed data:

```js
const multiply = (list, factor) => list.map(x => x * factor);
multiply([1, 2, 3], 5); // [5, 10, 15]
```

Note that the anonymous function `x => x * factor` is not pure.
Only `multiply` is a pure function.

### Memoization

> Memoization is a specific form of caching that involves caching the return value of a function based on its parameters.

The result of a pure function can be cached using its input as key and its invokation can be replaced by reading the result from the cache.
This performance optimization is one of the main reasons for using pure functions.

```js
const square = x => x * x;

const memoize = fn => {
  const cache = {};
  return arg => {
    console.log(cache);
    if (!(arg in cache)) {
      cache[arg] = fn(arg);
    }
    return cache[arg];
  };
};

const memoizedSquare = memoize(square);
memoizedSquare(5); // {}
memoizedSquare(5); // {5: 25}
```

Memoizing multiple parameters and keeping an entire history of those parameters quickly becomes a problem either in terms of time or memory.

In practice, memoization is either done only by using the first parameter as key (`lodash/memoize`) or by using a cache size of one (`reselect/createSelector`).
Either way, a single dimension array is enough to manage the cache.

### Side-effects

Any function that generates side-effects is considedred _impure_.

Here are a few examples of side-effects:

- printing to console
- setting a variable outside the function (eg: global variables)
- setting a cookie
- starting a HTTP request
- setting a timer
- forwarding an DOM event

A commercial application cannot be written using only pure functions.
User input, timers, asynchronous requests are all side-effects, making any function impure and usually forcing the imperative programming pattern.

> The goal of FP is to compose the majority of your program from small pieces of logic that can be combined together and reused.
> Side effects are inevitable but by limiting them to certain places in your application they will be easier to manage and track.
> (Source: [Hackernoon](https://hackernoon.com/functional-programming-paradigms-in-modern-javascript-pure-functions-797d9abbee1))

## Immutability

> The text-book definition of mutability is "liable or subject to change or alteration".
> In programming, we use the word to mean objects whose state is allowed to change over time.
> An immutable value is the exact opposite – after it has been created, it can never change.
> In plain English, it is "read-only".
> (Source: [SitePoint](https://www.sitepoint.com/immutability-javascript/))

### Native immutability in Javascript

In Javascript, primitive types (`string`, `number`, `boolean`, `null` and `undefined`) are immutable, while the non-primitive types (`function`, `object` and `Array`) are not.
To avoid mutation, changes to objects and arrays must be applied to a clone of the original.

```js
const bag = { apples: 3, oranges: 5 };
const newBag = { ...bag, apples: 5 }; // { apples: 5, oranges: 5 }
```

Developers applying the immutability principle must pay attention to the mutating array methods:

- `copyWithin`
- `fill`
- `pop`
- `push`
- `reverse`
- `shift`
- `sort`
- `splice`
- `unshift`

```js
const list = [1, 2, 3];
const reversed = [...list].reverse(); // [3, 2, 1]
console.log(list); // [1, 2, 3]
```

### Immutability for React

In a React app, immutability is a way of reducing computing time by relying on shallow comparison instead of deep comparison.

To make this work, the following rules must be applied:

1. create new objects or arrays only when they change
2. keep existing objects and arrays if no change is made

Consider this initial state where some parts of the app rely on `state.fruits`, while others on `state.sweets`. A change is applied by keeping all information that has not changed and overriding any properties that have changes:

```js
const state = {
  fruits: { apples: 3, oranges: 5, cherries: 0 },
  sweets: { icecream: 2, chocolate: 1 },
};
const updateChocolate = (state, amount) => ({
  ...state,
  sweets: {
    ...state.sweets,
    chocolate: amount,
  },
});
const newState = updateChocolate(state, 2);
console.log(state === newState); // false
console.log(state.fruits === newState.fruits); // true
console.log(state.sweets === newState.sweets); // false
```

After any deep change, all the parents of the deep field change.

In this case, `state`, `state.sweets` and `state.sweets.chocolate` fields are changed (new) while all others are strictly equal to the previous state.

The resulting state:

```json
{
  "fruits": { "apples": 3, "oranges": 5, "cherries": 0 },
  "sweets": { "icecream": 2, "chocolate": 2 }
}
```

The parts of the app that rely on `state.fruits` will not update, because `fruits` is the same object as before, while the others will update because `sweets` is a newly created object, indicating something inside it has changed.

## Declarative programming

Declarative programming is a pattern where code is written through **declarations**.

```js
const getFlag = ({ flag }) => flag;
```

For contrast, imperative programming is a pattern where code is written through **statements**.

```js
this.setState({});
```

> The main difference in functional programming in comparison to other programming paradigms is a declarative approach versus an imperative one.
> (Source: [Hackernoon](https://hackernoon.com/javascript-and-functional-programming-an-introduction-286aa625e26d))

Imperative programming is easily spotted by looking for functions whose returned value is not used.
In such case, it means the function generates side-effects, otherwise it would have no purpose.

### Javascript statements

Only the `return` statement is needed for declarative programming. `import` and `export` are also needed if working with ES6 modules.

Using any others is considered imperative programming:

`var` and `let` would need other statements for re-assignment.
Use `const` wherever a value declaration is needed.

`class` is not needed as there are no classes in functional programming.

`function` statement can be replaced by assigning an arrow function to a constant.

All iteration statements (`while`, `do...while`, `for`, `for...in`, `for...of`) can be replaced with array methods.

`if...else` statements can be replaced with the ternary operator.

```js
// Imperative programming
const getFee = isMember => {
  if (isMember) {
    return 2;
  } else {
    return 10;
  }
};

// Declarative programming
const getFee = isMember => (isMember ? 2 : 10);
```

For multiple `else` statements nested ternary operators can be used.

```js
// Imperative programming
const getFee = (isMember, isHappyHours) => {
  if (isMember) {
    return 2;
  } else if (isHappyHours) {
    return 5;
  } else {
    return 10;
  }
};

// Declarative programming
const getFee = (isMember, isHappyHours) =>
  isMember ? 2 : isHappyHours ? 5 : 10;
```

`switch` can be replaced with objects.

```js
// Imperative programming
const get = (type, a, b) => {
  switch (type) {
    case "first":
      return a;
    case "last":
      return b;
    default:
      return a + b;
  }
};

// Declarative programming
const typeFns = {
  first: a => a,
  last: (a, b) => b,
  default: (a, b) => a + b,
};
const get = (type, a, b) => (typeFns[type] || typeFns["default"])(a, b);
```

`break` and `continue` and `label` are not needed if loops and `switch` are not used.

`throw` and `try...catch` should not be used because `throw` creates side effects and using `try...catch` breaks predictability.

## Currying

> Currying is when we call a function with fewer arguments than it expects. In turn, the invoked function returns a function that takes the remaining arguments. (Source: [Hackernoon](https://hackernoon.com/javascript-and-functional-programming-currying-pt-4-96e3230782ab))

```js
const add = a => b => a + b;
const addFive = add(5);
const sum = addFive(10); // 15
```

The following function receives an array of books, a filtering flag and a search keyword, returning an array of matching books.

```js
const getMatches = (books, orderKey, keyword) =>
  [...books]
    .sort((a, b) => a[orderKey].localeCompare(b[orderKey]))
    .filter(book.title.includes(keyword) || book.author.includes(keyword));
getMatches([], "title", "a");
getMatches([], "title", "ab");
getMatches([], "title", "abc");
```

In a real-life situation, `orderKey` would be connected to a dropdown and `keyword` to a text input, while `books` would probably not change during the lifetime of a `Component`.

As the user types, the function gets called on every keystroke to filter the array of books in real-time.
On each call, the function first sorts the array by the selected order key, although `orderKey` doesn't change on keystrokes in the text input.

To eliminate redundant calculations (sorting in this example), we split the function into two: one for sorting and one for searching.
The sorting function is called when the sorting criteria changes and returns a searching function targeted only at the sorted array.

```js
const getMatchesCurried = (books, orderKey) => {
  const sortedBooks = [...books].sort((a, b) =>
    a[orderKey].localeCompare(b[orderKey])
  );
  return keyword =>
    sortedBooks.filter(
      book.title.includes(keyword) || book.author.includes(keyword)
    );
};

const sortedSearch = getMatchesCurried([], "title");
sortedSearch("a");
sortedSearch("ab");
sortedSearch("abc");
```

![](./jsperf-currying.png)
(Source: [JSPerf](https://jsperf.com/filter-currying))

## Composition

It is a pattern of combining multiple functions into one.
The result of each function is passed as the argument of the next, and the result of the last one is the result of the whole.

```js
const sum = (a, b) => a + b;
const square = x => x * x;
const isBig = x => x > 100;

const isBigSquareSum = (a, b) => isBig(square(sum(a, b)));
```

In the example above, the initial arguments are passed to the first function, `sum`.
The result is the passed to `square` and, finally, its result is passed to `isBig`.
The result of `isBig` is retuned as the result of the whole composition.
