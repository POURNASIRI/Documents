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