---
sidebar_position: 5
---

# `copyBuffer`

Copies the contents of one GPU buffer to another.

If the sizes/dimensions or data types of the two buffers don't match, an exception is thrown.

If a buffer name is used that wasn't specified in the `GPUInterface` constructor, an exception is thrown.

## Usage

```ts
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'theoriginal',
      size: [4],
      initialData: [9, 9, 9, 9],
    },
    {
      name: 'thecopy',
      size: [4],
    },
    {
      name: 'thethirdone',
      size: [60],
    },
  ],
});

await gpu.initialize();

gpu.copyBuffer('theoriginal', 'thecopy');

// error
// size mismatch
gpu.copyBuffer('theoriginal', 'thethirdone');
```
