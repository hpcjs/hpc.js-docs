---
sidebar_position: 4
---

# `readBuffer`

Reads a buffer from GPU memory into CPU memory, returning a `Float32Array`.

If a buffer name is used that wasn't specified in the `GPUInterface` constructor, an exception is thrown.

Note: this is an `async` method.

## Usage

```ts
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'result',
      size: [100],
    },
  ],
});

await gpu.initialize();

// calculate non-trivial zeroes of the zeta function

const buffer = await gpu.readBuffer('result');
const result = Array.from(buffer);

// do this if you're fancy
// not all targets support spread syntax though
const showoff = [...buffer];
```
