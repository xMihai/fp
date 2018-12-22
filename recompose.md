# recompose <!-- omit in toc -->

Recompose is a library that promotes functional programming of React components.
It does so using High Order Components, a concept derived from High Order Functions.
It allows to separate pieces of code used in one component in order to apply them to other components as well.

- [renderNothing](#rendernothing)
- [withProps](#withprops)

Part of [Functional Programming with React](./README.md) series.

## renderNothing

A HOC that returns a Component that renders nothing.

```js
const renderNothing = Component => () => null;
```

Moreover, this HOC is a pure function.
It is predictable because it always returns the same Component, regardless of parameters.

```js
const Nothing = () => null;

const renderNothing = Component => Nothing;
```

## withProps

Sets props on the given Component, overriding existing props with the same name.

The code of this HOC looks like this:

```js
const withProps = newProps => Component => props => (
  <Component {...{ ...props, ...newProps }} />
);
```

It is used to create component with specific configuration.
For example, a button that is alwyas disabled:

```js
const DisabledButton = withProps({ disabled: true })(Button);

// instead of...
const DisabledButton = props => <Button {...props} disabled />;
```

Note that props defined in `withProps` take precedence, beacuse they are spread after the props received by the final Component.
Setting `disabled` to `false` would have no effect.

```js
<DisabledButton disabled={false} />
```

`withProps` may also receive a callback instead of an options object.
The callback receives the current props as argument and must return the props to set.

The code of this HOC looks like this:

```js
const withProps = newPropsCallback => Component => props => (
  <Component
    {...{
      ...props,
      ...newPropsCallback(props),
    }}
  />
);
```

In the following example , `withProps` is used to calculate the missing `dirty` prop using a the existing `pristine` prop:

```js
const withDirtyFlag = withProps(({ pristine }) => ({ dirty: !pristine }));
const EnhancedField = withDirtyFlag(Field);

const EnhancedField = ({ pristine, ...rest }) => (
  <Field {...{ pristine, ...rest, dirty: !pristine }} />
);
```

```js
const withProps = newPropsOrGetter => Component => props => (
  <Component
    {...{
      ...props,
      ...(isFunction(newPropsOrGetter)
        ? newPropsOrGetter(props)
        : newPropsOrGetter),
    }}
  />
);
```
