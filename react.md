# react <!-- omit in toc -->

`react` is a library used to build a HTML DOM (Document Object Model) from input data.
Because rendering the DOM in browsers is an expensive operation, React became popular for the algorithm it uses to optimize re-renders.

- [Rendering in browser](#rendering-in-browser)
- [JSX](#jsx)
- [Components](#components)
- [Lifecycle methods](#lifecycle-methods)
  - [Mounting](#mounting)
  - [Updating](#updating)
  - [Unmounting](#unmounting)
- [Pure Components](#pure-components)
- [State](#state)
- [Context](#context)
- [Function components](#function-components)
- [High Order Components](#high-order-components)

Part of [Functional Programming for React](./README.md) series.

## Rendering in browser

In the following example, pure Javascript is used to dynamically create a DOM structure:

```js
const titleElement = document.createElement("h1");
const titleText = document.createTextNode("Hello World!");
titleElement.appendChild(titleText);

const paragraphElement = document.createElement("p");
const paragraphText = document.createTextNode("Lorem ipsum ...");
paragraphElement.appendChild(paragraphText);

const appElement = document.createElement("div");
appElement.setAttribute("id", "app");
appElement.appendChild(titleElement);
appElement.appendChild(paragraphElement);

document.getElementById("root").appendChild(appElement);
```

The resulting HTML:

```html
<div id="app">
  <h1>Hello World!</h1>
  <p>Lorem ipsum ...</p>
</div>
```

Next, the example is refactored using `React.createElement` instead of `document.createElement`:

```js
const titleElement = React.createElement("h1", {}, "Hello World!");
const paragraphElement = React.createElement("p", {}, "Lorem ipsum ...");
const appElement = React.createElement(
  "div",
  { id: "app" },
  titleElement,
  paragraphElement
);

ReactDOM.render(appElement, document.getElementById("root"));
```

Note that the elements created with `React.createElement` are not HTML elements, but entities that are in the end converted to HTML elements by `react-dom` library.

The structure created with `React.createElement` is called Virtual DOM.
Every time the attributes or children of an element are updated, they are compared to their previous values.
If there is no change, the browser DOM is not updated. Making this comparison is in most cases faster than re-rendering the DOM in browser.

## JSX

JSX is a Javascript extension that allows using HTML tags in Javascript code.
With the use of compilers like Babel or Typescript, JSX get is transformed to React elements.

```js
// JSX
const appElement = (
  <div id="app">
    <h1>Hello World!</h1>
    <p>Lorem ipsum ...</p>
  </div>
);

// plain JS
const appElement = React.createElement(
  "div",
  { id: "app" },
  React.createElement("h1", {}, "Hello World!"),
  React.createElement("p", {}, "Lorem ipsum ...")
);
```

Javscript expressions can be used in HTML tags by wrapping them in curly braces:

```js
// JSX
const sumElement = <div id={"Sum".toLowerCase()}>Sum is: {5 + 5}</div>;

// plain JS
const sumElement = React.createElement(
  "div",
  { id: "Sum".toLowerCase() },
  "Sum is: ",
  5 + 5
);
```

Because all attributes are collected in a plain object when translated, JSX supports a shorthand syntax for applying multiple attributes from a plain object.

```js
const attributes = { id: "title", style: "color: red" };
const titleElement = <div {...attributes}>Hello World!</div>;

// identical to
const titleElement = (
  <div id="title" style="color: red">
    Hello World!
  </div>
);
```

## Components

The atomic building blocks of React are `components`.

The simplest component is a function that receives _props_ and returns a React element or `null`. React element props are the equivalent of DOM element attributes. This kind of components are called _function components_.

```js
// JSX
const Title = ({ text }) => <h1>{text}</h1>;
const Paragraph = ({ text }) => <p>{text}</p>;
const App = () => (
  <div id="app">
    <Title text={"Hello World!"} />
    <Paragraph text={"Lorem ipsum ..."} />
  </div>
);
ReactDOM.render(<App />, document.getElementById("root"));

// plain JS, simplified

const Title = ({ text }) => React.createElement("h1", {}, text);
const Paragraph = ({ text }) => React.createElement("p", {}, text);
const App = () =>
  React.createElement(
    "div",
    { id: "app" },
    React.createElement(Title, { text: "Hello World!" }),
    React.createElement(Paragraph, { text: "Lorem ipsum ..." })
  );

ReactDOM.render(App({}), document.getElementById("root"));
```

> All React components must act like pure functions with respect to their props. (Source: [React Docs](https://reactjs.org/docs/components-and-props.html))

## Lifecycle methods

### Mounting

- constructor(props)
- render()
- componentDidMount()

### Updating

- shouldComponentUpdate(nextProps, nextState)
- render()
- componentDidUpdate(prevProps, prevState)

### Unmounting

- componentWillUnmount()

## Pure Components

`React.PureComponent` implements `shouldComponentUpdate` with a shallow prop and state comparison.
This stops the update on components whose props and state didn't change.

```js
shouldComponentUpdate(nextProps, nextState) {

}
```

## State

State can be set with `this.setState({})`.
The parameter must be an object, that will be merged internally to the current state object.

State can be used from `this.state`.

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { apples: 0 };
  }

  render() {
    const { apples } = this.state;
    return (
      <button {...{ onClick: () => this.setState({ apples: apples + 1 }) }}>
        {apples}
      </button>
    );
  }
}
```

## Context

Context is a state set by a component and read by all its children.
It uses a `Provider` and a `Consumer` to transfer a data object.

Consider the case where a prop needs to be passed down through multiple components:

```js
const App = () => <Section {...{ apples: 5 }} />;
const Section = ({ apples }) => <Toolbar {...{ apples: 5 }} />;
const Toolbar = ({ apples }) => <Card {...{ apples: 5 }} />;
const Card = ({ apples }) => <div>{apples}</div>;
```

Context creates a short path between components:

```js
const { Provider, Consumer } = React.createContext({ apples: 0 });

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { apples: 0 };
  }

  render() {
    const { apples } = this.state;
    return (
      <div>
        <Provider value={{ apples }}>
          <Section />
        </Provider>
        <button {...{ onClick: () => this.setState({ apples: apples + 1 }) }}>
          +
        </button>
      </div>
    );
  }
}

const Section = () => <Toolbar />;
const Toolbar = () => <Card />;
const Card = () => (
  <div>
    <Consumer>{({ apples }) => apples}</Consumer>
  </div>
);
```

## Function components

Function components are Javascript functions that receive a single argument (called props) and return a React element.

```js
const Paragraph = ({ text }) => <p>{text}</p>;
```

Function components don't have internal state or lifecycle methods.

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

## High Order Components

In functional programming, a _higher order function_ is a function that receives a function as argument or returns a function.

In React, a **higher order component** is a function that receives a component as argument and returns a component.
It is a concept similar to decorators in OOP, having the purpose of adding specific behavior to any component.

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
    <Component {...props} />
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
