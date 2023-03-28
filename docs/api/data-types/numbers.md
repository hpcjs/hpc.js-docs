---
sidebar_position: 1
---

# Numbers

Of course, the primary data type of HPC.js is the `number`. Vectors and arrays may be required for more complicated programs, but they all rely on the almighty floating-point format dictated by the IEEE 754 standard.

## Usage

Numbers behave the same way as they do in JavaScript. You can create variables and constants, use them as counters in for loops, index arrays with them, etc. To do math with them, use the `Math` library like usual.

```ts
const kernel = await gpu.createKernel(inputs => {
  const exponent = 2.5;
  let sum = 0;

  for (let i = 0; i < 10; i++) {
    sum += Math.pow(i, exponent);
  }
});
```

## Functions

Most functions from the `Math` library are supported. If you notice anything missing, it's most likely because it's not natively supported on the GPU and we didn't want to cause any confusion by implementing a software version.

The following functions are implemented in GPU code and hardware accelerated:

- `Math.abs(number)`
- `Math.acos(number)`
- `Math.acosh(number)`
- `Math.asin(number)`
- `Math.asinh(number)`
- `Math.atan(number)`
- `Math.atanh(number)`
- `Math.atan2(number, number)`
- `Math.ceil(number)`
- `Math.cos(number)`
- `Math.cosh(number)`
- `Math.exp(number)`
- `Math.floor(number)`
- `Math.log(number)`
- `Math.log2(number)`
- `Math.max(number)`
- `Math.min(number)`
- `Math.pow(number)`
- `Math.random()` (see [Randomness](../kernels/randomness))
- `Math.round(number)`
- `Math.sign(number)`
- `Math.sin(number)`
- `Math.sinh(number)`
- `Math.sqrt(number)`
- `Math.tan(number)`
- `Math.tanh(number)`
- `Math.trunc(number)`
