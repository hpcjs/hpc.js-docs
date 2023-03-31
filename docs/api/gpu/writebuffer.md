---
sidebar_position: 8
---

# `writeBuffer`

Writes the contents of an array to GPU memory.

:::warning

This method automatically runs in the background, which is why it isn't `async`. However, other operations (e.g. `GPUKernel.run`) will be forced to wait until this operation completes, so use this method sparingly, especially when dealing with large amounts of data. For more information, see our article on [asynchronous execution](../../best-practices/asynchrnous-execution).

:::

## Arguments

- `dst`
  - Type: `string`
- `data`
  - Type: `GPUBufferType[] | GPUBufferType[][] | GPUBufferType[][][]`
    - `GPUBufferType = number | vec2 | vec3 | vec4`

## Exceptions

- Invalid buffer name
- Data type mismatch
- Size mismatch

## Usage

```ts
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'randoms',
      size: [4],
    },
  ],
});

await gpu.initialize();

const data = new Array(4).fill(0).map(_ => Math.random());

gpu.writeBuffer('randoms', data);

// error
// size mismatch
gpu.copyBuffer('randoms', data.slice(1));
```
