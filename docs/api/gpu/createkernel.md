---
sidebar_position: 6
---

# `createKernel`

Given a JavaScript/TypeScript function, converts it into GPU code and returns a `GPUKernel` object that can be run with `GPUKernel.run`.

Accepts a single function parameter describing the kernel. Documentation on what can and can't be done in the kernel can be found [here](../kernels/inputs).

Note: this is an `async` method.

## Usage

```ts
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'ascending',
      size: [12],
    },
  ],
});

await gpu.initialize();

const kernel = await gpu.createKernel(inputs => {
  const index = inputs.threadId.x;
  inputs.buffers.ascending[index] = index;
});
kernel.run(12);
```
