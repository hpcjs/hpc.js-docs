---
sidebar_position: 1
---

# Getting Started

## Purpose

GPUs are absurdly powerful for the simple reason that they can perform thousands of operations in parallel, unlike traditional CPUs that are limited to sequential instructions. Unfortunately, programming them has never been easy (especially on the web), so with HPC.js we set out to address the most common pain points by:

- Creating a web-based interface, eliminating cross-platform concerns
- Using TypeScript to bridge main thread code and GPU code
- Abstracting away all unnecessary features and boilerplate

The 80-20 principle, the idea that we use 20% of features 80% of the time, is well-known among programmers, and nowhere is it more true than GPU programming, where it's really closer to 90-10. Our ultimate goal with HPC.js is to remove the hassle of low-level control so you can spend **less time writing boilerplate** and **more time getting stuff done.**

## Installation

Node:

```bash
npm install hpc.js
```

## Example Code

This guide is aimed at people who are unfamiliar with GPU compute, so if any part of this code looks foreign to you, head on over to [Introduction to GPU Compute](../gpu-compute-introduction). But, if you're already comfortable with JavaScript and existing GPU compute languages like CUDA, GLSL, or WGSL, this should tell you everything you need to know.

```ts
// Program that doubles each number in a GPU buffer

import GPUInterface from 'hpc.js';

const gpu = new GPUInterface({
  buffers: [{ name: 'mybuffer', size: [4], initialData: [1, 2, 3, 4] }],
});
await gpu.initialize();

const double = gpu.createKernel(inputs => {
  const index = inputs.threadId.x;
  inputs.buffers.mybuffer[index] *= 2;
});
double.run(4);

const result = await gpu.readBuffer('mybuffer');
console.log(result);
// Output: [2, 4, 6, 8]
```

## Resources

We have created a series of videos introducing HPC.js. The first is a quick 5-minute guide, and the second and third show how to make a substantial application (specifically a heat simulation) involving user input and rendering.
