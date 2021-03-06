# redux <!-- omit in toc -->

Redux is a state management library commonly used for React.
It keeps an application wide state that can be used directly by any component.

- [State](#state)
- [Updating](#updating)
- [Reducers and actions](#reducers-and-actions)
- [Connecting Redux to React](#connecting-redux-to-react)
- [Actions, action creators and handlers](#actions-action-creators-and-handlers)
- [Flux Standard Actions](#flux-standard-actions)
- [Selectors](#selectors)

Part of [Functional Programming for React](./README.md) series.

## State

State is a literal object kept inside the store.

```js
let state = {
  apples: 0,
  oranges: 2,
};
```

## Updating

The store is updated by calling the `dispatch` function with an `action`.

```js
// simplified version
const dispatch = action => {
  const newState = reducer(state, action);
  subscribers.forEach(subscriber => subscriber(newState));
};

dispatch({ type: "SET_APPLES", payload: 0 });
```

## Reducers and actions

Updating the state is done by calling a function called _reducer_.
The reducer expects two arguments, the current _state_ and a literal object called _action_, and returns the new state.

```js
// the simplest reducer
const reducer = (state, action) => ({
  ...state,
  ...action.payload,
});

const action = { payload: { apples: 1 } };

const newState = reducer(state, action); // { apples: 1, oranges: 2 }
```

Reducers must be pure functions.
They must be predictable and return the new state using only the current state and the provided action, without any external inputs.
They must also be free of side-effects.
A common mistake is triggering a second update from the reducer.

Because in practice actions must trigger operations much complex than a simple merge, the reducer will be composed of multiple reducers, each one performing a separate operation.

```js
const reducers = {
  SET_APPLES: (state, payload) => ({
    ...state,
    apples: payload,
  }),
  INCREMENT_APPLES: ({ apples, ...rest }, payload) => ({
    ...rest,
    apples: apples + payload,
  }),
  DEFAULT: state => state,
};

const reducer = (state, { type, payload }) =>
  (reducers[type] || reducers.DEFAULT)(state, payload);

const setApples = { type: "SET_APPLES", payload: 0 };
const addApples = { type: "INCREMENT_APPLES", payload: 2 };
```

Action creation should be decoupled from components for easy scalability.
Actions should be created through functions commonly called _action creators_.

```js
const setApples = apples => ({
  type: "SET_APPLES",
  payload: apples,
});
const resetApples = () => ({
  type: "SET_APPLES",
  payload: 0,
});
const initApples = resetApples;

// dispatch expects an action
dispatch(initApples());

// ..., not an action creator
dispatch(initApples);
```

## Connecting Redux to React

The connection is made with the `connect` function from the `react-redux` package.

`connect` receives as arguments two functions called `mapStateToProps` and `mapDispatchToProps`.

`mapStateToProps`, as the name suggests, it receives the Redux state and is expected to return a props object.

```js
const mapStateToProps = ({ apples, oranges }) => ({
  fruits: apples + oranges,
});
```

`mapDispatchToProps` receives the store's `dispatch` function and is expected to return a props object, each of the props being a function that when invoked, will call `dispatch` with an action.

```js
const mapDispatchToProps = dispatch => ({
  initFruits: () => {
    dispatch(initApples());
    dispatch(initOranges());
  },
});
```

`connect` will return a HOC that applies these mapped props on the given component.

```js
// simplified version
const connect = (mapStateToProps, mapDispatchToProps) => Component => ({
  store: { getState, dispatch },
  ...props
}) => (
  <Component
    {...{
      ...props,
      ...mapStateToProps(getState()),
      ...mapDispatchToProps(dispatch),
    }}
  />
);
```

## Actions, action creators and handlers

It is a common practice to have _action creators_ in a folder called _actions_, which fuels confusion regarding these terms for beginners.

Actions are literal objects:

```js
const resetAction = { type: "RESET" };
const updateAction = { type: "UPDATE", payload: { apples: 5 } };
```

Actions creators are functions that return actions. They may or may not receive arguments:

```js
const resetActionCreator = () => ({ type: "RESET" });
const updateActionCreator = payload => ({ type: "UPDATE", payload });
```

Handlers are functions that dispatch actions to redux:

```js
const resetHandler = () => dispatch(resetActionCreator());
const updateHandler = payload => dispatch(resetActionCreator(payload));
```

If `mapDispatchToProps` is given a literal object as argument, it will assume that each value is an action creator and will bind it to `dispatch` before setting them as props.
In the following example, `connect` will convert the action creators `reset` and `update` to handlers of the same name:

```js
connect(
  null,
  { reset, update }
);

// same as...
connect(
  null,
  dispatch => ({
    reset: () => dispatch(reset()),
    update: payload => dispatch(update(payload)),
  })
);
```

## Flux Standard Actions

_Flux Standard Actions_ is the most popular standard for creating actions.

An action must be a literal object and must always contain a property `type`.
This property must be of type `string`.

```js
const reset = () => ({ type: "RESET" });
```

The action may contain an optional property named `payload` that can be of any type.
This property holds data needed by the _reducer_ to make the state update.

```js
const reset = () => ({ type: "SET_APPLES", payload: 0 });
const setApples = apples => ({
  type: "SET_APPLES",
  payload: apples,
});
```

The action may contain an optional property named `error` of type `boolean`.
When this property is set to `true`, the `payload` property must contain an `Error` instance.

```js
const resetError = message => ({
  type: "SET_APPLES",
  error: true,
  payload: new Error(message),
});
```

The action may contain an optional property named `meta` that can be of any type.

```js
const fetchFromApi = () => ({
  type: "FETCH",
  meta: { url: "/resource" },
});
```

## Selectors

`mapStateToProps` is also called a _selector_.

`mapStateToProps` and selectors in general receive the current props as second argument.

```js
const mapStateToProps = ({ apples, oranges }, { selectedFruit }) => ({
  selected: { apples, oranges }[selectedFruit] || 0,
  total: apples + oranges,
});
```

In practice, Redux state is a deeply nested objectand many operations require data from diffrent branches.
Selectors are used to split operations into reusable pieces.

```js
const state = {
  bag: {
    fruits: {
      apples: 0,
      oranges: 2,
    },
  },
};

// selectors
const getFruits = state => state.bag.fruits;
const getTotal = state => {
  const { apples, oranges } = getFruits(state);
  return apples + oranges;
};
const getSelected = (state, props) =>
  getFruits(state)[props.selectedFruits] || 0;

const mapStateToProps = (state, props) => ({
  selected: getSelected(state, props),
  total: getTotal(state),
});
```
