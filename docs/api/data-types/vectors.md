---
sidebar_position: 1
---

# Vectors

HPC.js supports the `vec2`, `vec3`, and `vec4` types commonly found in GPU programming environments. These can be used to concisely represent multidimensional data, including:

- Thread ID
- Pixel position
- Physical position
- Color values

Additionally, GPUs often come with dedicated arithmetic processors for these data types, meaning you could get a 4x speedup by using `vec4` instead of `number`!

## Constructors

To create a `vecN` type on either the CPU or GPU, simply import `vec2`, `vec3`, or `vec4` from `hpc.js`. `vecN` has two constructors: one where you specify each component, and one where you specify a single number to be written to each component.

```ts
import { vec3 } from 'hpc.js';

const vec1 = vec3(1, 2, 3);
const vec2 = vec3(0);
```

## Vector components

- `vec2` has components `x`, `y`
- `vec3` has components `x`, `y`, `z`
- `vec4` has components `x`, `y`, `z`, `w`

## Vector Arithmetic

Because JavaScript doesn't support operator overloading, we provided member functions to make arithmetic easy. `vecN.plus(vecN)`, for example. A full list of relevant methods is provided at the bottom of the page.

```ts
import { vec2 } from 'hpc.js';

const a = vec3(1, 2, 3);
const b = vec3(4, 5, 6);
const sum = a.plus(b);
```

When in doubt, assume that vector functions operate on a per-element basis. `vecN.div(vecN)`, for example, returns a new `vecN` that looks like `vecN(a.x/b.x, a.y/b.y, ...)`. This applies to more exotic operators like `sin(vecN)` and `atan2(vecN, vecN)` as well.

Vectors also support the mathematical operations we expect, including `vecN.dot(vecN)`, `vec3.cross(vec3)`, `vecN.length()`, and `vecN.normalized()`.

## Swizzling

Swizzling is a surprisingly useful feature that may seem odd at first, but in fact lets us write code that's faster and easier to read. It's best illustrated by example:

```ts
import { vec4 } from 'hpc.js';

const a = vec4(1, 2, 3, 4);
const b = a.zyy;
// b = vec3(4, 3, 3);
```

Swizzling is most commonly used to extract 'sub-vectors' out of larger vectors, or construct larger vectors out of smaller ones. You will often find something like this at the beginning of HPC.js kernels:

```ts
const kernel = await gpu.createKernel(inputs => {
  const pixel = threadId.xy;
  // pixel is a vec2

  ...
})

kernel.run(canvas.width, canvas.height);
```

You can get pretty creative with it. Nothing is off-limits, as long as your swizzle is between 2 and 4 characters long and only accesses existing components.

```ts
const a = vec2(1, 2);

// allowed
a.xyyx;
a.yyy;

// not allowed: a.z and a.w don't exist
a.zwzw;
```

## Vector Functions

Math
