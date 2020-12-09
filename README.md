# notes-react-hooks

## Why React Hooks?

## `useState`

for creating functional components with state.

## `useEffect`

It takes the place of the following lifecycle methods:

- `componentDidMount`
- `componendDidUpdate`
- `componentWillUnmount`

You have to declare the **dependency array** to avoid innecesary runs (the values it depends on), so if any of the declared values changes, the code inside the `useEffect` hook will execute again, otherwise, it won't.

## `useReducer`

## `useMemo`

## `useCallback`

## Custom Hooks

To keep your hooks logic _DRY_, you can extract the repeating patterns to a new component and set the local state there (with `useState`), then returning an array containing `[state, setState, <optional>]`
