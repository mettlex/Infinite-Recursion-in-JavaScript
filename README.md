# Infinite Recursion in JavaScript

Suppose you have the following recursive function:

```js
function count(i, max) {
  if (i === max) return i;

  return count(i + 1, max);
}
```

Call the count function with `i = 1`, `max = 100000` and it'll give you an error like **InternalError: too much recursion** or **RangeError: Maximum call stack size exceeded** in most JavaScript runtimes. Try running this on Firefox or Chromium-based browsers:

```js
count(1, 100_000)
```

> InternalError: too much recursion

The call stack is limited and the JavaScript engines on [Firefox (SpiderMonkey)](https://bugzilla.mozilla.org/show_bug.cgi?id=723959) & [Chrome (V8)](https://chromestatus.com/feature/5516876633341952) unlike [Safari (WebKit)](https://webkit.org/blog/6240/ecmascript-6-proper-tail-calls-in-webkit/) don't support proper tail call elimination of the ES2015 specification as of March 2023. For most JavaScript engines, when a function calls itself many times, it results in a stack overflow error if the number of recursive calls exceeds the maximum size of the call stack.

> _You might be asking: "What can be done to avoid overflowing the call stack?"_

The solution is to give the control back to the Event Loop by using techniques such as asynchronous callbacks or promises. This allows the call stack to be cleared between recursive calls, preventing it from exceeding its maximum size.

**One way to do this is to make two simple changes to your code:**

1. Make the function asynchronous by adding the `async` keyword for the function.
2. Add `await Promise.resolve()` line just before the recursive function call.

Let's see how it works for our previous example:

```js
// 1. add async keyword
async function countAsync(i, max) {
  if (i === max) return i;

  // 2. add the following line
  await Promise.resolve();
  
  return countAsync(i + 1, max);
}
```

Now let's try calling the async function with the same arguments as before:

```js
await countAsync(1, 100_000)
```

> 100000

**This time it doesn't throw an error!**

Let's use this same technique to calculate big Fibonacci number. Also, let's use modern Arrow function syntax this time:

```js
const fibonacci = async (n) => {
  const fibo = async (a, b, n) => {
    if (n < 1) {
      return b;
    }

    await Promise.resolve();

    return fibo(b, a + b, n - 1);
  };

  return fibo(BigInt(0), BigInt(1), n - 1);
}
```

```js
await fibonacci(50_000) // gives a big integer
```

**Happy Coding!**

\- Mettle X
