---
sidebar_position: 1
---

# Inputs

Kernel functions must have exactly one input object. This object contains information relevant to your kernel and compute environment, including:

- The kernel invocation ID (aka thread ID)
- Buffer data and uniform data
- Canvas dimensions and pixel plotting

and more!

In our examples, we call this object `inputs`, but you can call it anything you want. We suggest naming it `g` if you're going for terse code.

## `threadId`

This is a `vec3` indicating the ID of the current kernel thread. Since GPU kernels may run millions of times, we need a way to differentiate each invocation, and the thread ID allows us to make this distinction. The components of `inputs.threadId` range from 0 up to, but not including, the number specified in `kernel.run(x, y, z)`. If a number isn't specified, it defaults to 0.

### Example

```ts
const kernel = await gpu.createKernel(inputs => {
  const x = inputs.threadId.x;
  const y = inputs.threadId.y;
  const z = inputs.threadId.z;
  const sum = x + y + z;
});
kernel.run(3, 5);

// x ranges from 0-2
// y ranges from 0-4
// z is always 0
```

## `buffers`

This object contains references to the buffers specified in the `GPUInterface` constructor. Their shape matches the original description, so if we say `mybuffer` is of type `number[][]`, we can't say `mybuffer[x]` or `mybuffer[x][y][z]`; the kernel will only compile with `mybuffer[x][y]`. Additionally, these objects cannot be reassigned, so don't attempt to make a copy of your buffer with `const copy = mybuffer`.

:::info

Because JavaScript does not differentiate between floating point numbers and integers, we allow you to index buffers using floats. When you do so, the index will simply be rounded down to an integer.

:::

### Example

```ts
const kernel = await gpu.createKernel(inputs => {
  const x = Math.random() * 5;
  const y = Math.random() * 5;
  const result = inputs.buffers.data[x][y];
});
```

### Buffer Size

To get the size of a buffer, simply use the `dim` function imported from `hpc.js`. It'll return a `number`, `vec2`, or `vec3` depending on the size of the buffer. This is also the one place where you _are_ allowed to partially index a buffer: if you have a buffer of size `[x, y, z]`, `dim(inputs.buffers.mybuffer[0])` will return `vec2(y, z)`.

## `uniforms`

This object contains references to the uniforms specified in the `GPUInterface` constructor. These variables are the same for all threads within a single `kernel.run()` call, so they tend to be more "universal". For example, we might include mouse position, timestamp, or keys pressed as uniforms. While it would be possible to use buffers alone and avoid uniforms, changing a uniform is much faster than changing a buffer, so if you're frequently updating variables from the CPU, look to use a uniform.

Because uniforms are, well, _uniform_ across all threads, they cannot be changed from within the kernel. They must be updated through `GPUInterface.setUniforms()`.

### Example

```ts
const kernel = await gpu.createKernel(inputs => {
  const time = inputs.uniforms.milliseconds;
  const mousePressed = inputs.uniforms.mousePressed;

  if (mousePressed && time < 1000) {
    // do something!
  }
});
```

## `canvas`

This object contains information and functions relevant to the `canvas` passed to the `GPUInterface` constructor. After your kernel(s) have rendered to the canvas to your satisfaction, you can call `gpu.updateCanvas()` to see your masterpiece.

### `size`

A `vec2` containing the size of the canvas.

### `setPixel`

`setPixel(x: number, y: number, r: number, g: number, b: number)`  
`setPixel(pos: vec2, r: number, g: number, b: number)`  
`setPixel(x: number, y: number, color: vec3)`  
`setPixel(pos: vec2, color: vec3)`

Allows the kernel to set a pixel. `x` and `y` must be within the canvas size, and `r`, `g`, and `b` must be within the range `0-255`, inclusive. Non-integer values will be rounded down.

You may notice there are four overloads for this method. Choose whichever best suits the form of your data; they all work the same in the end.

### Example

```ts
const kernel = await gpu.createKernel(inputs => {
  const pixel = inputs.threadId.xy;
  const size = inputs.canvas.size;

  const r = (pixel.x / size.x) * 255;
  const g = (pixel.y / size.y) * 255;
  const b = 0;
  inputs.canvas.setPixel(pixel, r, g, b);
});

kernel.run(canvas.width, canvas.height);
```

## `usingCpu`

HPC.js supports automatic CPU fallback, but since CPUs are considerably slower than GPUs, we may want to run a simplified version of our code in case a GPU can't be acquired. `usingCpu` is a boolean that tells us this inside the kernel.

### Example

```ts
const kernel = await gpu.createKernel(inputs => {
  let sum = 0;

  if (inputs.usingCpu) {
    for (let i = 0; i < 100; i++) {
      sum += i;
    }
  } else {
    for (let i = 0; i < 10000; i++) {
      sum += i;
    }
  }
});
```
