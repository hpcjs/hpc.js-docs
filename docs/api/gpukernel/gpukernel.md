# `GPUKernel`

The `GPUKernel` class represents a handle on a specific kernel, a function that runs on the GPU. These can only be created using [`GPUInterface.createKernel`](../gpu/createkernel.md) like so:

```ts
const gpu = new GPUInterface({
    ...
});
await gpu.initialize();

const kernel = await gpu.createKernel(inputs => {
    ...
});
```

## Properties

### `source`

Contains the transpiled code as a result of `GPUInterface.createKernel`. This generally isn't useful, but it comes in handy for submitting bug reports or debugging an issue while waiting for a fix to come through.

## Methods

### `run`

Runs the GPU kernel this object is associated with. [Documentation here](run).
