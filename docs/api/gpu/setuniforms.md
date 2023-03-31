---
sidebar_position: 3
---

# `setUniforms`

Updates uniform variables specified at `GPUInterface` creation. Uniforms can be of type `number`, `boolean`, `vec2`, `vec3`, or `vec4`.

## Arguments

- `uniforms`
  - Type: `Record<string, number | vec2 | vec3 | vec4 | boolean>`

## Exceptions

- Invalid uniform name
- Incorrect uniform type

## Usage

```ts
const gpu = new GPUInterface({
  uniforms: {
    call: 3,
    me: vec2(4, 5),
    maybe: true,
  },
});

await gpu.initialize();

gpu.setUniforms({
  call: -9001,
  me: vec2(1, 2),
});

// error
// not specified in the constructor
gpu.setUniforms({
  blondie: 1980,
});

// error
// type mismatch
gpu.setUniforms({
  me: false,
});
```
