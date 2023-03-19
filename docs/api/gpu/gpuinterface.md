---
sidebar_position: 1
---

# `GPUInterface`

The main export of HPC.js. `GPUInterface` is an abstraction over WebGPU that makes working with the GPU far easier.

## Parameters

The `GPUInterface` constructor accepts a configuration object parameter structured as follows:

```ts
type GPUBufferSize = [number] | [number, number] | [number, number, number];
type GPUBufferData = number[] | number[][] | number[][][];

type GPUInterfaceOptions = {
  buffers?: { name: string; size: GPUBufferSize; initialData?: GPUBufferData };
  uniforms?: { [key: string]: number };
  canvas?: HTMLCanvasElement;
  useCpu?: boolean;
};
```

Each parameter is optional, and different applications may call for different combinations. It's possible you may only need a canvas and uniforms, with no buffers.

Note: the type definitions above are simplified for readability. In fact, TypeScript will prevent you from mismatching `GPUBufferSize` and `GPUBufferData` dimension, and an exception will be thrown if this is caught at runtime.

### `buffers`

This parameter specifies the buffers we want to create on the GPU.

`name` exists purely within HPC.js. It allows you to reference the buffer from within the kernel.

`size` specifies the size of the buffer. It can be a 1-tuple, 2-tuple, or 3-tuple, depending on whether you want your data to be 1D, 2D, or 3D.

`initialData` allows you to optionally load data into your buffer at its creation, which is very common. If you want to write data after creation, you can use `GPUInterface.writeBuffer`.

### `uniforms`

This parameter specifies the uniform variables accessible to our kernels.

Each kernel invocation typically works on a different piece of data from a given buffer, but uniform variables are the same across all threads, hence "uniform". They can be set with `GPUInterface.setUniforms`.

### `canvas`

This parameter lets you pass a canvas to the `GPUInterface`, enabling pixel rendering.

If the canvas is provided, the `inputs.funcs.setPixel` function becomes available from within the kernel, as well as `GPUInterface.updateCanvas`. With these tools, you can use GPU compute to create magnificent, dynamic illustrations.

### `useCpu`

This parameter lets you control whether your kernel code runs on the CPU by default. This is more for performance testing and debugging, as it makes little sense to use a GPU acceleration library for CPU code! But since HPC.js offers automatic CPU fallback, it's important to test.

## Usage

```ts
import GPUInterface, { vec2 } from 'hpc.js';

const thecanvas = document.querySelector('canvas')!;
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'lotterynumbers',
      size: [6],
      initialData: [63, 58, 90, 22, 46, 15],
    },
    {
      name: 'tetrispiece',
      size: [2, 3],
      initialData: [
        [1, 0, 0],
        [1, 1, 1],
      ],
    },
  ],
  uniforms: {
    mousePos: vec2(0, 0),
    temperature: 30,
  },
  canvas: thecanvas,
});
```

## Properties

### `isInitialized`

Stores whether or not the `GPUInterface` has been initialized.

### `backendType`

Stores the backend type of the `GPUInterface`. If uninitialized, this returns `'uninitialized'`. Otherwise it returns `'gpu` if a GPU was found and `'cpu'` if the CPU fallback was enabled or if `useCpu: true` was selected in the constructor.

## Methods

### `initialize`

**Must be called before any other methods.**

Prepares the `GPUInterface` and required resources on the GPU. [Documentation here](initialize).

### `createKernel`

Converts a given JavaScript/TypeScript function into GPU code. [Documentation here](createkernel).

### `setUniforms`

Accepts an object containing any combination of uniform variables, and writes them to the GPU. [Documentation here](setuniforms).

### `readBuffer`

Reads a GPU buffer into CPU memory. [Documentation here](readbuffer).

### `copyBuffer`

Copies the contents of one GPU buffer into another. [Documentation here](copybuffer).

### `updateCanvas`

Paints the canvas with the pixels specified by `inputs.canvas.setPixel` from kernel code. [Documentation here](updatecanvas)
