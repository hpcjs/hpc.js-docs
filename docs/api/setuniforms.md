---
sidebar_position: 3
---

# `setUniforms`

Updates uniform variables specified at `GPUInterface` creation. Currently only accepts numbers for updated values.

If a uniform name is included that wasn't specified in the constructor, an exception is thrown.

## Usage

```ts
const gpu = new GPUInterface({
  uniforms: {
    call: 3,
    me: 4,
    maybe: 5,
  },
});

await gpu.initialize();

gpu.setUniforms({
  call: 100,
  me: 42,
});

// error
// not specified in the constructor
gpu.setUniforms({
  blondie: 1980,
});
```
