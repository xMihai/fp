# recompose

Recompose is a library that promotes functional programming of ReactJS components.
It does so using High Order Components, a concept derived from High Order Functions.
It allows to separate pieces of code used in one component in order to apply them to other components as well.

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

```js
const DisabledButton = withProps({ disabled: true })(Button);

const DisabledButton = props => <Button {...props} disabled />;
```

The code of this HOC looks like this:

```js
const withProps = newProps => Component => props => (
  <Component {...{ ...props, ...newProps }} />
);
```

Note that props defined in `withProps` take precedence, beacuse they are spread after the props received by the final Component.

`withProps` may also receive a function that returns props.

```js
const EnhancedField = withProps(({ pristine }) => ({ dirty: !pristine }))(
  Field
);

const EnhancedField = ({ pristine, ...rest }) => (
  <Field {...{ pristine, ...rest, dirty: !pristine }} />
);
```

The code of this HOC looks like this:

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
