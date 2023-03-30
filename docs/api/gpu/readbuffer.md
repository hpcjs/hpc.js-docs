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

## Packing

When transferring data from the GPU to the CPU, HPC.js has to find a way to pack the GPU buffer data into a `Float32Array`. For `number`s this works just as we'd expect, but it also works with `vecN` buffers. In the case of `vec2` and `vec4`, components are packed sequentially, so

```ts
[vec2(1, 2), vec2(3, 4)];

// becomes

new Float32Array([1, 2, 3, 4]);
```

`vec3` packing is a bit different: WebGPU, the API behind HPC.js, packs `vec3`s like `vec4`s, so the fourth component will always be a zero:

```ts
[vec3(1, 2, 3), vec3(4, 5, 6)];

// becomes

new Float32Array([1, 2, 3, 0, 4, 5, 6, 0]);
```
