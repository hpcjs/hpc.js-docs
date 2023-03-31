# `run`

Runs the GPU kernel this object is associated with. The components of `inputs.threadId` will range according to the specified dimensions, with undefined dimensions defaulting to 1.

## Parameters

- `x`
  - Type: `number`
- `y`
  - Type: `number | undefined`
- `z`
  - Type: `number | undefined`

## Exceptions

- Any arguments are not a positive integer

## Usage

```ts
const kernel = await gpu.createKernel({
  // do something
});

// z is inferred to be 1
kernel.run(3, 2);
```
