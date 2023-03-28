---
sidebar_position: 3
---

# Arrays

We don't need to motivate arrays for you; everyone knows why they're useful. They're fully present in HPC.js, but they're not quite as free-form as they are in JavaScript. Specifically, they're not objects, so they don't have methods like `map` and `push`.

Instead, arrays are constant-size blocks of contiguous memory. They can hold `number`, `boolean`, `vec2`, `vec3`, or `vec4`. Array types can't be mixed, so everything inside it must be of the same type.

## Constructor

Arrays are constructed either by passing a JavaScript array literal to the `array` function imported from `hpc.js`, or by passing a `size` and `fill` argument.

:::caution

The `array(size, fill)` constructor copies the `fill` argument before evaluating it, much like a C-style macro. If you don't want the argument to be evaluated multiple times, like an expensive function call, save it to a variable and pass that instead!

:::

```ts
import { array } from 'hpc.js';

const kernel = await gpu.createKernel(inputs => {
  // first method
  const primaryColors = array([
    vec3(255, 0, 0),
    vec3(0, 255, 0),
    vec3(0, 0, 255),
  ]);

  // second method
  // this array is 10 numbers long, filled with 0
  let squares = array(10, 0);
  for (let i = 0; i < 10; i++) {
    squares[i] = i ** 2;
  }
});
```

## Array length

To get the length of an array, pass it into the `dim` function (short for 'dimension'), imported from `hpc.js`.

```ts
import { array, vec3 } from 'hpc.js';

const kernel = await gpu.createKernel(inputs => {
  const colors = array([
    vec3(66, 30, 15),
    vec3(9, 1, 47),
    vec3(0, 7, 100),
    vec3(24, 82, 177),
    vec3(134, 181, 229),
    vec3(241, 233, 191),
    vec3(255, 170, 0),
    vec3(153, 87, 0),
  ]);

  const length = dim(colors);
  // length = 8
});
```

## Constant arrays

When an array in HPC.js is marked as `const`, its elements cannot be modified. This may be unfamiliar coming from JavaScript, but just remember to apply the same principle as always: make it `const` unless it needs to be `let`.

```ts
import { array, vec3 } from 'hpc.js';

const kernel = await gpu.createKernel(inputs => {
  const arr1 = array(16, vec3(0, 0, 0));
  let arr2 = array(16, vec3(0, 0, 0));

  // not allowed
  arr1[0] = vec3(1, 2, 3);

  // perfectly fine
  arr2[0] = vec3(1, 2, 3);
});
```
