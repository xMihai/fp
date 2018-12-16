# Review of ES6 features <!-- omit in toc -->

- [ECMAScript](#ecmascript)
- [ES updates](#es-updates)
- [Let and Const](#let-and-const)
- [Arrow functions](#arrow-functions)
- [Template literals](#template-literals)
- [Shorthand object property assignment](#shorthand-object-property-assignment)
- [Default parameters](#default-parameters)
- [Destructuring assignments](#destructuring-assignments)
- [Spread operator](#spread-operator)
- [Rest operator](#rest-operator)

Part of [Functional Programming with ReactJS](./README.md) series.

## ECMAScript

> ECMAScript (or ES) is a trademarked scripting-language specification standardized by Ecma International in ECMA-262 and ISO/IEC 16262.
> It was **created to standardize JavaScript**, so as to foster multiple independent implementations.
> JavaScript has remained the best-known implementation of ECMAScript since the standard was first published, with other well-known implementations including JScript and ActionScript. (Source: [Wikipedia](https://en.wikipedia.org/wiki/ECMAScript))

## ES updates

ES6 is the most significant update to the language, and the first update to the language since ES5 was standardized in 2009.

ES6 was released in 2015 and is also known as ES2015.
At the time of this release it was announced that the language will receive yearly updates.
The latest update is ES9 released in 2018.

The features in these updates take some time before they are implemented in browser and reach the market.
To overcome this, tools like Babel or compilers like TypeScript will parse ES6+ code and transform it to ES5 so browsers can run it.

## Let and Const

`let` is the new `var`.

Variables declared by `let` have their scope in the block for which they are defined, as well as in any contained sub-blocks.
In this way, `let` works very much like `var`.
The main difference is that the scope of a variable declared with `var` is the entire enclosing function.

```js
function example() {
  if (true) {
    var a = 1;
    let b = 2;
  }
  console.log(a); // 1
  console.log(b); // ReferenceError: b is not defined
}
```

Reference: [MDN: let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

Constants are block-scoped, much like variables defined using the let statement.
The value of a constant cannot change through reassignment, and it can't be redeclared.
Objects assigned to constants can be changed, but a new object cannot be assigned to an existing constant.
The reason behind this is that the value of a variable or constant holding an object is actually a memory pointer to the object.
The object itself is not read-only, but the value assigned to the constant is.

```js
const bag = { apples: 3 };
bag.apples++;
console.log(bag); // { apples: 4 }

bag = { oranges: 5 }; // TypeError: Assignment to constant variable.
```

Reference: [MDN: const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)

## Arrow functions

An arrow function expression has a shorter syntax than a function expression and does not have its own `this`, `arguments`, `super`, or `new.target`.
These function expressions are best suited for non-method functions, and they cannot be used as constructors.

<!-- prettier-ignore-start -->
```js
// ES5
function square(x) {
  return x * x;
}

// ES6
const square = (x) => x * x;
```
<!-- prettier-ignore-end -->

For single arguments, parentheses can be omitted.
The following two statements are both correct.

```js
const square = x => x * x;
```

For multiple or no parameters, parentheses are required.

```js
const sum = (a, b) => a + b;
const zero = () => 0;
```

Arrow functions can have either a "concise body" (examples above) or the usual "block body".

```js
const division = (a, b) => {
  if (b == 0) return NaN;
  return a / b;
};
```

To return an object literal, wrap it in parentheses to differentiate object syntax from block syntax.

```js
const square = x => ({ square: x * x });
```

Reference: [MDN: arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## Template literals

Template literals are string literals using back tick instead of single or double quotes.

Template literals can span multiple lines.

```js
const es5 = "string text line 1\n" + "string text line 2";
const es6 = `string text line 1
string text line 2`;
```

Template literals support interpolation of any kind of expression.

```js
const revenue = 10;
const costs = 7;
const es5 = "Profit is" + (revenue - costs) + " Euro";
const es6 = `Profit is ${revenue - costs} Euro`;
```

Reference: [MDN: template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## Shorthand object property assignment

The shorthand syntax can be used when a variable is assigned to an object property of the same name.

```js
const apples = 1;

// ES5
var bag = { apples: apples };

// ES6
const bag = { apples };
```

Reference: [MDN: template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## Default parameters

Default function parameters allow named parameters to be initialized with default values if no value or undefined is passed.

In ES5, the general strategy for setting defaults was to test parameter values in the function body and assign a value if they are undefined.

```js
// ES5
function getCount(list) {
  if (typeof list === "undefined") {
    list = [];
  }
  return list.length;
}

// ES6
const getCount = (list = []) => list.length;
```

Default parameters can be used with destructuring assignment. Note that the default parameter is used only when the parameter is `undefined`.

```js
// ES5
function getCount(bag) {
  if (typeof bag === "undefined") {
    bag = { apples: 0 };
  }
  return bag.apples;
}

// ES6
const getApples = ({ apples } = { apples: 0 }) => apples;
getApples(); // 0
getApples({}); // undefined
```

Reference: [MDN: default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)

## Destructuring assignments

The destructuring assignment syntax is a JavaScript expression that makes it possible to unpack values from arrays, or properties from objects, into distinct variables.

Destructuring arrays:

```js
const list = ["a", "b", "c"];

// ES5
var first = list[0];
var second = list[1];

// ES6
const [first, second] = list;
```

Destructuring objects:

```js
const bag = { apples: 2 };

// ES5
var apples = bag.apples;

// ES6
const { apples } = bag;
```

Destructuring arguments:

```js
// ES5
function getApples(bag) {
  return bag.apples;
}

// ES6
const getApples = ({ apples }) => apples;
```

Destructuring nested values:

<!-- prettier-ignore-start -->
```js
const bag = { fruits: { apples: 5 } };

// ES5
var apples = bag.fruits.apples;

// ES6
const { fruits: { apples } } = bag;
```
<!-- prettier-ignore-end -->

Reference: [MDN: destructuring assignments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

## Spread operator

The spread operator (`...`) allows an array (or string) to be expanded as function arguments.

```js
const list = [5, 8, 2, 6];

// ES5
var highest = Math.max.apply(null, list);

// ES6
const highest = Math.max(...list);
```

The spread operator can expand array elements into another array, performing a clone or merge.

```js
const fruits = ["apples", "oranges", "bananas"];
const drinks = ["soda", "wine"];

// ES5
var fruitsClone = fruits.slice(0);
var groceries = fruits.concat(drinks);

// ES6
const fruitsClone = [...fruits];
const groceries = [...fruits, ...drinks];
```

When used on an object, the key-value pairs can be expanded into other objects, performing a clone or merge.

```js
const bag = { apples: 3, oranges: 5 };

// ES5
var bagClone = Object.assign({}, bag);
var bigBag = Object.assign({}, bag, otherBag);

// ES6
const bagClone = { ...bag };
const bigBag = { ...bag, ...otherBag };
```

Note that spread operator does not do deep cloning. It only takes items (or property values) and puts them in a new array (or object).

Reference: [MDN: spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)

## Rest operator

The rest operator has the same notation as the spread operator (`...`).
A function's last parameter can be prefixed with the rest operator which will cause all remaining arguments to be placed within an array.
Only the last parameter can be a _rest_ parameter.

```js
const check = (...args) => console.log(args);
check(1, 2, 3); // [1, 2, 3]

const rest = (first, ...others) => console.log(others);
rest(1, 2, 3); // [2, 3]
rest(1); // []
```

The rest operator can be used with destructuring assignment to gather remaining elements into an array:

```js
const list = [1, 2, 3];
const [first, ...others] = list;
console.log(others); // [2, 3]
```

It can also be used to gather remaining properties from a destructured object.

```js
const bag = { apples: 3, oranges: 5, cherries: 0 };
const { oranges, ...others } = bag;
console.log(others); // { apples: 3, cherries: 0 }}
```

Reference: [MDN: rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
