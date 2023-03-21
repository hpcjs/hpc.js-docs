---
sidebar_position: 2
---

# Race Conditions

Race conditions are a category of bugs that occur when two threads write to a variable at the same time. They are extremely prevalent in parallel programming and should always be at the forefront of your mind. It's important to understand them well, as it's very possible that you've never encountered them in your programming career thus far.

## Introduction

To illustrate, consider the following example:

```ts
const gpu = new GPUInterface({
  buffers: [
    {
      name: 'data',
      size: [1],
    },
  ],
});
await gpu.initialize();

const kernel = await gpu.createKernel(inputs => {
  const x = inputs.threadId.x;
  inputs.buffers.data[0] = x;
});
kernel.run(256);

const result = await gpu.readBuffer('data');
console.log(result[0]);
```

What number is logged here? We have 256 threads all writing to the same number, but their order of execution is unclear. We can guess that the last one to run will have its number stored as the final result, but can we be sure?

Now, imagine we used this kernel instead:

```ts
const kernel = await gpu.createKernel(inputs => {
  const current = inputs.buffers.data[0];
  const x = inputs.threadId.x;

  if (current % 5 === 0) {
    inputs.buffers.data[0] = x;
  }
});
kernel.run(256);
```

This is even worse! Not only does the final output depend on the execution order, but now the behavior of each thread depends on the threads that ran before it.

## Considerations and Solutions

Race conditions are often identified by getting different results each time you run your program, and can occur for a number of reasons. Sometimes, their existence indicates a fundamental flaw in the design of a program, in which case the only cure is to step back and rethink your code's execution path.

Other times, they can be solved by adding a level of redundancy to your code. For example, in the [Heat Simulation Tutorial](../tutorials/heat-simulation), we avoided a race condition by writing our results to a temporary buffer and copying that buffer back to the original. These steps can seem bothersome and slow, and sometimes they truly are unnecessary; we recommend benchmarking your code to weigh the benfits of properly handing a race condition.

There's no catch-all solution to solving race conditions, making them one of the most frustrating bugs to deal with as a parallel programmer. As always, when you believe you have a race condition on your hands, reduce your code down to the simplest possible example to isolate the problem.
