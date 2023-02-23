---
sidebar_position: 1.5
---

# Kernel Code

`GPUInterface.createKernel` converts JavaScript/TypeScript code into a GPU kernel, but not all language features are supported. In practice, this hardly matters, since the main purpose of GPU compute is to accelerate math.

## Features

Not all JavaScript functions are supported. So, what are we allowed to do?

- Control flow (if/else, for, while, switch, do while)
- Variable declaration and assignment
- Common operations
  - Arithmetic: `+ - * / % **`
  - Comparisons: `== != > < >= <=`
  - Ternary: `? :`
- Most functions from the `Math` library, including `Math.random`

What you can't do:

- Use string variables (these may be supported in a future version)
- Directly include external variables (if you want to do this, use a uniform) and functions
- Creating arrays, and array methods like `map`
- Objects, including destructuring syntax with `inputs`
- Browser interaction functions, like anything under `document`, obviously
- `console.log` (and other `console` methods)

Note: if the absence of `console.log` is alarming, we suggest creating a `debug` buffer, writing to it, and then logging it from outside the kernel with `readBuffer`.

GPU compute is almost always used to speed up math operations, which are entirely supported. Besides that, it makes no sense to do most of these operations in a GPU kernel, so it's not at all an issue. For example, `Array.map` is a really nice function to use, but passing a function and then repeatedly applying it is definitely much slower than just writing a for loop.

## Examples

Good usage:

```ts
// this kernel doesn't do anything useful
// but all of the syntax is valid
const kernel = gpu.createKernel(inputs => {
  const index = inputs.threadId.x;
  const number1 = inputs.buffers.data1[index];
  const number2 = inputs.buffers.data2[index];

  let combined;
  if (number2 > 0) {
    combined = Math.floor(number1 * math.exp(number2));
  } else {
    let sum = 0;

    for (let i = 0; i < 5; i++) {
      sum += Math.log(number1) % (number2 + i);
    }

    combined = sum;
  }

  inputs.buffers.output = combined;
});
```

Bad usage:

```ts
const oingo = 'boingo';

// this contains like ten errors
const kernel = gpu.createKernel(inputs => {
  const { x: index } = inputs.threadId;

  const arr = new Array(x).fill(0).map({_, i} => i);
  console.log(arr);

  const [text, setText] = React.useState(oingo);
  const span = document.querySelector('span');
  span.innerText = state;
});
```
