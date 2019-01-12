# Functional React <!-- omit in toc -->

- [Function components](#function-components)
- [Spreading props in JSX](#spreading-props-in-jsx)
- [High Order Components](#high-order-components)
  - [Wrapper HOCs](#wrapper-hocs)

Part of [Functional Programming with React](./README.md) series.

## Function components

Function components are Javascript functions that receive a single argument (called props) and return a React element.

```js
const Paragraph = ({ text }) => <p>{text}</p>;
```

Function components don't have internal state or lifecycle methods.

> All React components must act like pure functions with respect to their props. (Source: [React Docs](https://reactjs.org/docs/components-and-props.html))

There is no difference in terms of performance between function and class components.
Internally, React wraps function components in class components.

```js
// simplified version
class ActualParagraph extends React.Component {
  render() {
    Paragraph(this.props);
  }
}
```

(Source: [Component Rendering Performance in React](https://medium.com/modus-create-front-end-development/component-rendering-performance-in-react-df859b474adc))

## Spreading props in JSX

In many instances, props need to be passed down to other components.

In the following example the `input` element is rendered wrapped in a `div` block but must receive relevant attribute values.

```js
const WrappedInput = ({ value, disabled }) => (
  <div>
    <input value={value} disabled={disabled} />
  </div>
);
```

With this syntax, every time a new prop is needed it has to be manually added on the `input` element.
Or, we can spread the `props` object, each of it fields becoming an attribute on the inner component.

```js
const WrappedInput = props => (
  <div>
    <input {...props} />
  </div>
);
```

But this syntax only supports spreading a single object.
To spread multiple objects, or add props, they must be first gathered in a new object that is spread as props.

```js
const WrappedInput = props => (
  <div>
    <input {...{ ...props, disabled: true }} />
  </div>
);
```

## High Order Components

In functional programming, a _higher order function_ is a function that receives a function as argument or returns a function.

In React, a **higher order component** is a function that receives a component as argument and returns a component.
It is a concept similar to decorators in OOP, having the purpose of adding specific behavior to any component.

### Wrapper HOCs

One of the simplest and most common HOCs are the ones that wrap components in other elements.

For example, wrapping all input fields of a form in `td` elements can be shortly done with a HOC:

```js
const inTableCell = Component => props => (
  <td>
    <Component {...props} />
  </td>
);

const NameInputCell = inTableCell(NameInput);
const EmailInputCell = inTableCell(EmailInput);

// instead of...
const NameInputCell = props => (
  <td>
    <NameInput {...props} />
  </td>
);

const EmailInputCell = props => (
  <td>
    <EmailInput {...props} />
  </td>
);
```

HOCs can also receive options by currying two arguments, an `options` object and a `Component`.

```js
const inTableCell = ({ span }) => Component => props => (
  <td colSpan={span}>
    <Component {...{ ...props, disabled: onlyDisplay }} />
  </td>
);

const inNarrowCell = inTableCell({ span: 1 });
const inWideCell = inTableCell({ span: 3 });

const AddressInputCell = inWideCell(AddressInput);

// instead of...
const AddressInputCell = props => (
  <td colSpan={3}>
    <AddressInput {...props} />
  </td>
);
```
