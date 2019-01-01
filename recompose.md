# recompose <!-- omit in toc -->

Recompose is a library that promotes functional programming of React components.
It does so using High Order Components, a concept derived from High Order Functions.
It allows to separate pieces of code used in one component in order to apply them to other components as well.

- [Know your functions](#know-your-functions)
- [Props manipulation](#props-manipulation)
  - [withProps](#withprops)
  - [defaultProps](#defaultprops)
  - [mapProps](#mapprops)
  - [renameProp](#renameprop)
  - [renameProps](#renameprops)
- [Handlers](#handlers)
  - [withHandlers](#withhandlers)
  - [Optimization of handlers](#optimization-of-handlers)
- [State](#state)
  - [withState](#withstate)
  - [withStateHandlers](#withstatehandlers)
- [Composition](#composition)
  - [compose](#compose)
  - [Order of composition](#order-of-composition)
- [Other utilities](#other-utilities)
  - [nest](#nest)
  - [renderComponent](#rendercomponent)
  - [renderNothing](#rendernothing)
  - [branch](#branch)

Part of [Functional Programming with React](./README.md) series.

## Know your functions

Components are functions that receive props as argument.
They start with `props =>`.

```js
const Input = props => <div>{props.text}</div>;
```

Higher Order Components are functions that receive a Component as argument and return a Component.
They start with `Component => props =>`.

```js
const wrapper = Component => props => <Component>{props.text}</Component>;
```

Recompose functions are not HOCs, but functions that always return a HOC.
They start with: `options => Component => props =>`.

```js
const withEnhancement = options => Component => props => (
  <Component>{options.flag ? props.text : props.number}</Component>
);
```

## Props manipulation

### withProps

`withProps` produces a HOC that sets new props on the given Component by merging them with the current props.
The new props take precedence over the current props.

```js
// simplified version
const withProps = newProps => Component => props => (
  <Component {...{ ...props, ...newProps }} />
);
```

In the following example, `withProps` is used to create a button that is alwyas disabled:

```js
const DisabledButton = withProps({ disabled: true })(Button);

// instead of...
const DisabledButton = props => <Button {...props} disabled />;
```

Remember that new props take precedence, beacuse they are spread after the props received by the final Component.
Setting `disabled` to `false` would have no effect.

```js
<DisabledButton disabled={false} />
```

`withProps` may also receive a callback instead of an options object.
The callback receives the current props as argument and must return the new props.
New props will take precendece over current props when merged.

```js
// simplified version
const withProps = newPropsCallback => Component => props => (
  <Component
    {...{
      ...props,
      ...newPropsCallback(props),
    }}
  />
);
```

In the following example , `withProps` is used to calculate the missing `dirty` prop using a the current `pristine` prop:

```js
const withDirtyFlag = withProps(({ pristine }) => ({ dirty: !pristine }));
const EnhancedField = withDirtyFlag(Field);

const EnhancedField = ({ pristine, ...rest }) => (
  <Field {...{ pristine, ...rest, dirty: !pristine }} />
);
```

### defaultProps

`defaultProps` is a variation of `withProps` where current props take precedence over the default props.

```js
// simplified version
const defaultProps = defaults => Component => props => (
  <Component {...{ ...defaults, ...props }} />
);
```

In the following example, `defaultProps` is used to create a button disabled by default, but that can be enabled by explicitly setting disabled to `false`.

```js
const DisabledButton = defaultProps({ disabled: true })(Button);

// instead of...
const DisabledButton = props => <Button {...{ disabled: true, ...props }} />;
```

`defaultProps` can only receive an object as argument.

### mapProps

`mapProps` is a variation of `withProps` where current props are replaced with the new props.

It receives a callback as argument.
The callback receives the current props as argument and must return the replacement props.

```js
// simplified version
const mapProps = newPropsCallback => Component => props => (
  <Component {...newPropsCallback(props)} />
);
```

In the following examle, `mapProps` is used to remove all current props and only set the `dirty` prop:

```js
const withDirtyFlag = mapProps(({ pristine }) => ({ dirty: !pristine }));
const EnhancedField = withDirtyFlag(Field);

// instead of
const EnhancedField = ({ pristine }) => <Component {...{ dirty: !pristine }} />;
```

### renameProp

`renameProp` produces a HOC that renames one prop.

```js
// simplified version
const renameProp = (oldName, newName) => Component => props => (
  <Component
    {...Object.keys(props).reduce(
      (result, key) => ({
        ...result,
        [key === oldName ? newName : key]: props[key],
      }),
      {}
    )}
  />
);
```

In the following example, `renameProp` is used to comply to another component's API:

```js
const fromUsers = renameProp("users", "items");
const UsersList = fromUsers(List);
```

### renameProps

`renameProps` produces a HOC that renames multiple props.

```js
// simplified version
const renameProps = nameMap => {
  const oldNames = Object.keys(nameMap);
  return Component => props => (
    <Component
      {...Object.keys(props).reduce(
        (result, key) => ({
          ...result,
          [oldNames.includes(key) ? nameMap[key] : key]: props[key],
        }),
        {}
      )}
    />
  );
};
```

## Handlers

Handlers are props that handle side-effects.
If we consider usual props creating a upward flow of data from app state to DOM elements, then handlers create an downward flow to app state.
All side effects must be encapsulated in handlers to create a better separation of concerns.

`redux`' action creators are one example of handlers.

### withHandlers

`withHandlers` produces a HOC that adds handlers as props.

In practice, it's role is to combine existing handlers or to manipulate data before using an existing handler.
Access to existing props is achieved by using handle creators, higher order functions that receive props as argument and return an actual handler.

```js
// simplified version
const withHandlers = handlerCreators => Component => props => {
  const handlers = Object.entries(handlerCreators).reduce(
    (result, [name, handlerCreator]) => ({
      ...result,
      [name]: handlerCreator(props),
    }),
    {}
  );
  return <Component {...{ ...props, ...handlers }} />;
};
```

In the following example, `withHandlers` is used to pull the value of a field from the DOM event before calling another handler:

```js
const EnhancedInput = withHandlers({
  onChange: ({ updateState }) => event => updateState(event.target.value),
})(Input);

// instead of
const EnhancedInput = props => (
  <Input
    {...{ ...props, onChange: event => props.updateState(event.target.value) }}
  />
);
```

Note that a handler creator is a handler function transformed in a pure function by also providing all dependencies as argument.

```js
const handler = event => updateState(event.target.value);
const handlerCreator = ({ updateState }) => event =>
  updateState(event.target.value);
```

### Optimization of handlers

Because in Javascript functions are first class objects, handlers can be assigned as props using `withProps`:

```js
// withHandlers
const EnhancedInput = withHandlers({
  onChange: ({ updateState }) => event => updateState(event.target.value),
})(Input);

// withProps
const EnhancedInputs = withProps(({ updateState }) => ({
  onChange: event => updateState(event.target.value),
}))(Input);
```

On each React update, the callback of `withProps` will be called and `onChange` will be a new function every time.
Unless `updateState` function changes, this is redundant.
`withHandlers` solves this by using a non-FP technique.

```js
// simplified version
const withHandlers = handlerCreators => Component => {
  class WithHandlers extends React.Component {
    // pre-defined handlers that use `this.props`
    handlers = handlerCreators.map(handlerCreator => (...args) =>
      handlerCreator(this.props)(...args)
    );

    render() {
      return <Component {...{ ...this.props, ...this.handlers }} />;
    }
  }
  return WithHandlers;
};
```

It creates pre-defined handlers that access `props` from `this` object.
Handlers are by definition impure because they create side-effects.
Using `this` object also makes them unpredictable.

From the perspective of handlers function programming, `withHandlers` is better than `withProps` because it does not add any overhead on updates and separates handlers from props.

## State

### withState

`withState` produces a HOC that emulates component state for wrapped function components.
It sets two new props, one holding the value and the other being the handler that updates the state.

```js
// simplified version
const withState = (stateName, stateUpdaterName, initialState) => Component => {
  let state = intialState;
  const stateHandler = value => {
    state = value;
  };

  return props => (
    <Component
      {...{ ...props, [stateName]: state, [stateUpdaterName]: stateHandler }}
    />
  );
};
```

In the following example, `withState` is used to create a click counter:

```js
const withCounter = withState("count", "setCount", 0);

const Button = ({ count, setCount }) => (
  <button {...{ onClick: () => setCount(count + 1) }}>{count}</button>
);

const CounterButton = withCounter(button);
```

### withStateHandlers

`withStateHandlers` is a variation of `withState` that holds multiple values and uses custom handlers instead of simple setters.

In the following example, `withState` is used to create a click counter in both directions:

```js
const withCounter = withStateHandlers(
  { count: 0 },
  {
    increment: ({ count }) => () => ({ count: count + 1 }),
    decrement: ({ count }) => () => ({ count: count - 1 }),
  }
);

const Controls = ({ count, increment, decrement }) => (
  <div>
    {count}
    <button onClick={increment}>+</button>
    <button onClick={decrement}>-</button>
  </div>
);

const Counter = withCounter(Controls);
```

Note that the state retuned by the handlers is merged into the state object.
This is contrast with the behavior of `withState` setter that replaces the old value.

## Composition

It is a pattern of combining multiple HOCs into one.
The result of each HOC is passed as the argument of the next, and the last result the result of the whole.

### compose

`compose` is an utility that allows combining multiple HOCs with a readble syntax.

```js
const compose = (...hocs) => Component =>
  [...hocs]
    .reverse()
    .reduce((ComposedComponent, hoc) => hoc(ComposedComponent), Component);
```

In the example below, `compose` is used to combine `withState` and `withHandlers` to create a HOC to the effect of `withStateHandlers`:

```js
const withCounterState = withState("count", "setCount", 0);
const withCounterHandlers = withHandlers({
  increment: ({ count, setCount }) => () => setCount(count + 1),
  decrement: ({ count, setCount }) => () => setCount(count - 1),
});

const Counter = compose(
  withCounterState,
  withCounterHandlers
)(Controls);

// instead of...
const Counter = withCounterState(withCounterHandlers(Controls));
```

### Order of composition

`compose` expects arguments in mathematical order.

In the example below, `Input` is given as argument to the last function, `withC`.
The final component is the one resulting from `withA`.

```js
const EnhancedInput = withA(withB(withC(Input)));

const EnhancedInput = compose(
  withA,
  withB,
  withC
)(Input);
```

Let's consider all HOCs are wrappers:

```js
const withA = Component => props => (
  <A>
    <Component {...props} />
  </A>
);
const withB = Component => props => (
  <B>
    <Component {...props} />
  </B>
);
const withC = Component => props => (
  <C>
    <Component {...props} />
  </C>
);

const EnhancedInput = compose(
  withA,
  withB,
  withC
)(Input);

// instead of...
const EnhancedInput = props => (
  <A>
    <B>
      <C>
        <Input {...props} />
      </C>
    </B>
  </A>
);
```

This bottom to top composition makes the `props` flow more intuitive.
Props created by `withB` will be available in `withC`, but not in `withA`.

## Other utilities

### nest

`nest` expects any number of Components and produces a HOC that will use these Components to wrap the input Component.

```js
const EnhancedInput = nest(A, B, C)(Input);

// instead of...
const EnhancedInput = props => (
  <A {...props}>
    <B {...props}>
      <C {...props}>
        <Input {...props} />
      </C>
    </B>
  </A>
);
```

It is similar to `compose`, but expects components instead of HOCs.
In the example above, `nest` will first wrap `Input` with the last argument component, `C`.
Everything is wrapped with the first argument component, `A` in this case.

### renderComponent

`renderComponent` expects a Component and produces a HOC that always returns this Component.

```js
const renderComponent => Component => () => props => <Component {...props} />
```

### renderNothing

`renderNothing` produces a HOC that always returns a Component that renders nothing.
The HOC is pure as it always returns the same Component.

```js
const Nothing = () => null;
const renderNothing = () => () => Nothing;
```

### branch

`branch` expects three arguments:

- a function that return a boolean
- a HOC for the `true` case
- an optional HOC for the `false` case

It produces a HOC that passes the input Component through either of the `true` or `false` HOCs.

```js
const branch = (condition, hocTrue, hocFalse) => Component => props => {
  const Branch = condition(props)
    ? hocTrue(Component)
    : hocFalse
    ? hocFalse(Component)
    : Component;
  return <Branch {...props} />;
};
```

If the `false` HOC is omitted and the condition evaluates to this case, the input Component is retuned.

In the example below, `branch` is used with `renderNothing` to conditionally render `Input`:

```js
const EnhancedComponent = branch(({ hide }) => hide, renderNothing())(Input);
```

When `hide` is `true`, nothing is rendered.
Because the second HOC is not given, when `hide` is `false` Input will be rendered directly.
