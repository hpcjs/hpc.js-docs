---
sidebar_position: 5
---

# `copyBuffer`

Copies the contents of one GPU buffer to another.

## Arguments

- `src`
  - Type: `string`
- `dst`
  - Type: `string`

## Exceptions

- Invalid buffer name
- Data type mismatch
- Size mismatch

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
