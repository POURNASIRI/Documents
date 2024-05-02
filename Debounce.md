## What is debounce?

`Debounce` is a function that waits for a certain amount of time before executing a function.
Debouncing is a strategy used to improve the `performance` of a feature by controlling the time at which a function should be executed.

Debouncing accepts a function and transforms it in to an updated (debounced) function so that the code inside the original function is executed after a certain period of time.If the debounced function is called again within that period, the previous timer is reset and a new timer is started for this function call. The process repeats for each function call.

## How to Implement Debouncing in JavaScript?

#### What behavior do we want from the debounced function?

- Delay the function execution by a certain time, `delay`.
- Reset the timer if the function is called again.

To debounce a function, we'll have a separate function that accepts the function reference and the delay as parameters, and returns a debounced function.

```js
function debounce(func, delay) {
      return () => {}   // return debounced function
}
```
This function will only be called once to return a debounced function and that, in turn, will be used in the subsequent code.

```js
function debounce(func, delay) {
    return () => {
        setTimeout(() => {
        	func()
        }, delay)
    }
}
```
This `delays` the function call by delay milliseconds. But this is incomplete as it only satisfies the first requirement. How do we achieve the second behaviour?
Let's create a variable `timeout` and assign it to the return value of `setTimeout` method. The `setTimeout` method returns a unique identifier to the `timeout`, which is held by `timeout` variable.

```js
function debounce(func, delay) {
    let timeout=null
    return () => {
        timeout=setTimeout(() => {
            func()
        }, delay)
    }
}
```

Each time you invoke `setTimeout`, the ID is different. We will use this `timeout` variable to reset the timer.

But how do we get access to `timeout` from outside the debounce() method? As mentioned before, debounce() is only used once to return a debounced function. This, in turn, performs the debouncing logic.

Then, how does the debounced function have access to `timeout` even if it is used outside the debounce() function? Well, it uses a concept called closure.




## React Hooks

### useMemo

`useMemo` is a React Hook that lets you cache the result of a calculation between re-renders.

```js
const cachedValue = useMemo(calculateValue, dependencies)
```
Call `useMemo` at the top level of your component to cache a calculation between re-renders:

```js
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```
### Parameters

- `calculateValue`: The function calculating the value that you want to cache. It should be pure, should take no arguments, and should return a value of any type. React will call your function during the initial render. On next renders, React will return the same value again if the `dependencies` have not changed since the last render. Otherwise, it will call `calculateValue`, return its result, and store it so it can be reused later.
- `dependencies`: The list of all reactive values referenced inside of the `calculateValue` code. Reactive values include props, state, and all the variables and functions declared directly inside your component body.

### Return Value
- On the initial render, `useMemo` returns the result of calling `calculateValue` with no arguments.
During next renders, it will either return an already stored value from the last render (if the dependencies haven’t changed), or call `calculateValue` again, and return the result that `calculateValue` has returned.

## Usage 

### Skipping expensive recalculations
To cache a calculation between re-renders, wrap it in a `useMemo` call at the top level of your component:
```js
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

You need to pass two things to `useMemo`:

- A `calculation function` that takes no arguments, like `() =>`, and returns what you wanted to calculate.
- A `list of dependencies` including every value within your component that’s used inside your calculation.

### Skipping re-rendering of components 

In some cases, `useMemo` can also help you optimize performance of re-rendering child components. To illustrate this, let’s say this `TodoList` component passes the `visibleTodos` as a prop to the child `List` component:

```js
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

You’ve noticed that toggling the `theme` prop freezes the app for a moment, but if you remove `<List /> `from your JSX, it feels fast. This tells you that it’s worth trying to optimize the ``List`` component.

**By default, when a component re-renders, React re-renders all of its children recursively**. This is why, when `TodoList` re-renders with a different `theme`, the `List` component also re-renders. This is fine for components that don’t require much calculation to re-render. But if you’ve verified that a re-render is slow, you can tell `List` to skip re-rendering when its props are the same as on last render by wrapping it in `memo`:

```js
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

**With this change, `List` will skip re-rendering if all of its props are the same as on the last render**. This is where caching the calculation becomes important! Imagine that you calculated `visibleTodos` without `useMemo`:

```js
export default function TodoList({ todos, tab, theme }) {
  // Every time the theme changes, this will be a different array...
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ... so List's props will never be the same, and it will re-render every time */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**In the above example, the `filterTodos` function always creates a different array**, similar to how the {} object literal always creates a new object. Normally, this wouldn’t be a problem, but it means that List props will never be the same, and your memo optimization won’t work. This is where useMemo comes in handy:


```js
export default function TodoList({ todos, tab, theme }) {
  // Tell React to cache your calculation between re-renders...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...so as long as these dependencies don't change...
  );
  return (
    <div className={theme}>
      {/* ...List will receive the same props and can skip re-rendering */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**By wrapping the `visibleTodos` calculation in ``useMemo``, you ensure that it has the same value between the re-renders (until dependencies change). You don’t have to wrap a calculation in `useMemo` unless you do it for some specific reason.** In this example, the reason is that you pass it to a component wrapped in `memo`, and this lets it skip re-rendering. There are a few other reasons to add `useMemo` which are described further on this page.