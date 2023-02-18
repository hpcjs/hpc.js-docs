---
sidebar_position: 1
---

# Your First Program

:::info

This tutorial is available in video form at

:::

## The Plan

This tutorial is going to cover writing a basic program with HPC.js. We're going to create a list of numbers `[1, 2, 3, 4]`, double each one, and then log it to the console. Ensure you have HPC.js installed before proceeding!

## Initialization

First, we import the library.

```ts
import GPUInterface from 'hpc.js';
```

Everything in HPC.js runs through the `GPUInterface` class. It's our way of communicating with the GPU without having to slog through all the low-level protocols of WebGPU. To instantiate it, we simply call it's constructor.

```ts
const gpu = new GPUInterface({
  buffers: [{ name: 'mybuffer', initialData: [1, 2, 3, 4], size: [4] }],
});
```

Woah! A lot just happened there, so let's break it down.

When we perform GPU compute, data is typically passed through arrays, or buffers. This is perfect for the massively parallel model of computing that GPUs support, since these buffers can hold a lot of data, and each thread can operate on a different section of the buffer.. When we create the GPUInterface, we need to specify which buffers we want to create, so let's write down what we know:

- We want to create one buffer
- We need to give it a name, like `'mybuffer'`
- We need to fill it with data -- in our case, `[1, 2, 3, 4]`
- We need to give it a size -- our buffer is one-dimensional and four numbers long, so its size is `[4]`

Because we only have one buffer, our list is one buffer long. We pass all of this information to the `GPUInterface` constructor, and voilà! We have our interface.

Next, we need to run the `initialize` method on our `GPUInterface`.

```ts
await gpu.initialize();
```

Essentially, this method prepares all of the resources on the GPU that we requested, like our buffer. If we try to do anything on the GPU before calling `initialize`, the program will error. We only need to call this method once, but since we need to do it before anything else, it makes sense to run it soon after the constructor.

:::caution gpu initilization

Make sure you run `gpu.initialize()` before doing anything with your `GPUInterface`, or your program will crash!

:::

Now, let's get to writing some GPU code!

## Writing our GPU Code

In GPU computing, a function that runs on the GPU is known as a _kernel_. Creating a kernel with HPC.js is incredibly simple.

```ts
const kernel = gpu.createKernel(/* kernel code */);
```

And to run the kernel:

```ts
kernel.run(4);
```

Easy! But how do we write the kernel code? Well, the power of HPC.js is that you can continue to use JavaScript or TypeScript to write your GPU code! In other environments, such as CUDA or HLSL, you would have to switch to an entirely different programming language. By removing the need to do so, we've made this process remarkably less time-consuming.

---

We want to write a kernel that doubles a single number in the list, and then runs that kernel once for each number. `createKernel` accepts kernel code in the form of a function, like so:

```ts
const kernel = gpu.createKernel(inputs => {
  /* kernel code */
});
```

The `inputs` object contains information relevant to our kernel. We care about two things in this object, `inputs.buffers` and `inputs.threadId`. The `inputs.buffers` object gives us references to the buffers we specified in the constructor, namely `'mybuffer'`. And since our kernel will be executed four times, the `inputs.threadId` object tells us which ID our invocation is assigned.

```ts
const kernel = gpu.createKernel(inputs => {
  const index = inputs.threadId.x;
  inputs.buffers.mybuffer[index] *= 2;
});
```

There are two operations performed in this kernel. First, we take the `x` component out of `inputs.threadId` and assign it to a new variable, `index`. Why use `x`? Think of the buffer as a line going from left to right. As `x` increases, we move further to the right, pointing to a specific item in the buffer. Next, we use `index` to select a number from `inputs.buffers.mybuffer` and double it, exactly like normal JavaScript.

This kernel will double a single number in the list, so to double all numbers, we need to run it four times.

```ts
kernel.run(4);
```

The `4` here corresponds to `threadId.x`. If you're working with a 2D or 3D array, you can specify `kernel.run(x, y)` or `kernel.run(x, y, z)`.

At this point, our entire program looks like this:

```ts
import GPUInterface from 'hpc.js';

const gpu = new GPUInterface({
  buffers: [{ name: 'mybuffer', initialData: [1, 2, 3, 4], size: [4] }],
});
await gpu.initialize();

const kernel = gpu.createKernel(inputs => {
  const index = inputs.threadId.x;
  inputs.buffers.mybuffer[index] *= 2;
});
kernel.run(4);
```

That's actually all we need to double our numbers! In just 10 lines of code, we've written a GPU program that would have taken over 100 lines and lots of configuration to get working in other languages. However, we still need to get our numbers back and log them, so let's bring this home.

## Getting our Data Back

Reading a buffer from the GPU is as simple as

```ts
const result = await gpu.readBuffer('mybuffer');
```

This gives us a `Float32Array`, but we can easily turn this into a normal array with `Array.from(result)`. To print it, all we need is

```ts
console.log(Array.from(result));
```

Thus, our final program:

```ts
import GPUInterface from 'hpc.js';

const gpu = new GPUInterface({
  buffers: [{ name: 'mybuffer', initialData: [1, 2, 3, 4], size: [4] }],
});
await gpu.initialize();

const kernel = gpu.createKernel(inputs => {
  const index = inputs.threadId.x;
  inputs.buffers.mybuffer[index] *= 2;
});
kernel.run(4);

const result = await gpu.readBuffer('mybuffer');
console.log(Array.from(result));
```

:::danger unmapping your buffers

When you read a buffer using `gpu.readBuffer`, the buffer is put into a _mapped_ state. In this state, the buffer is readable by our main program, but we can't do GPU operations on it. Once you're done reading a buffer, you should call `gpu.unmapBuffer` to return it to its normal, unmapped state.

:::

## Conclusion

And that's it! In 12 lines of code, we've written a fully functional GPU program. From here, we encourage you the begin tinkering with the library and creating more substantial projects.

If you want to learn more advanced features of HPC.js, we suggest watching our tutorial on creating a heat simulation [here](/). In this video, we cover user interactivity, graphics, and uniform variables, so be sure to check it out!
