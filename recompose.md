# recompose <!-- omit in toc -->

Recompose is a library that promotes functional programming of React components.
It does so using High Order Components, a concept derived from High Order Functions.
It allows to separate pieces of code used in one component in order to apply them to other components as well.

- [withProps](#withprops)
- [defaultProps](#defaultprops)
- [mapProps](#mapprops)

Part of [Functional Programming with React](./README.md) series.

## withProps

`withProps` is a HOC that sets new props on the given Component by merging them with the current props.
The new props take precedence over the current props.

The code of this HOC looks like this:

```js
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

In the following example , `withProps` is used to calculate the missing `dirty` prop using a the current `pristine` prop:

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

## defaultProps

`defaultProps` is a variation of `withProps` where current props take precedence over the default props.

The code of this HOC looks like this:

```js
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

## mapProps

`mapProps` is a variation of `withProps` where current props are replaced with the new props.

It receives a callback as argument.
The callback receives the current props as argument and must return the replacement props.

The code of this HOC looks like this:

```js
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
