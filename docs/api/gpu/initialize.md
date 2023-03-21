---
sidebar_position: 2
---

# `initialize`

Sets up resources on the GPU used by `GPUInterface`. Accepts no parameters.

This function _must_ be called before any other methods on `GPUInterface`. If this rule is violated, an exception will be thrown.

Note: this method is `async`.

## Usage

```ts
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'johnnytest',
      size: [8],
    },
  ],
});

// note the absence of method calls

await gpu.initialize();

// go crazy
```

## Chaining

`GPUInterface.initialize()` returns a reference to `this`, meaning you can chain it into the constructor for shorter code. If this syntax seems bizarre, note that `new` binds tighter than member access in JavaScript, so the execution order makes sense.

```ts
const gpu = await new GPUInterface({ ... }).initialize();
```
