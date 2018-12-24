# recompose <!-- omit in toc -->

Recompose is a library that promotes functional programming of React components.
It does so using High Order Components, a concept derived from High Order Functions.
It allows to separate pieces of code used in one component in order to apply them to other components as well.

- [withProps](#withprops)
- [defaultProps](#defaultprops)
- [mapProps](#mapprops)
- [renameProp](#renameprop)
- [renameProps](#renameprops)

Part of [Functional Programming with React](./README.md) series.

## withProps

`withProps` is a HOC that sets new props on the given Component by merging them with the current props.
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

## defaultProps

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

## mapProps

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

## renameProp

`renameProp` is HOC that renames one prop.

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

## renameProps

`renameProps` is HOC that renames multiple props.

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
