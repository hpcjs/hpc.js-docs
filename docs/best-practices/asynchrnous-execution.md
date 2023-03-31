---
sidebar_position: 4
---

# Asynchronous Execution

If you haven't noticed by now, our selection of which methods to make `async` may seem a bit sporadic. However, these choices were made deliberately, and understanding them will allow you to write more performant GPU code. To understand why the `async` methods are what they are, we first need to go over command queues, and to understand command queues, we need to see how HPC.js actually works.

## The Magic behind HPC.js

To harness the power of the GPU, HPC.js internally uses a cutting-edge API called WebGPU. In the past, similar libraries have used WebGL, which doesn't natively support GPU computation and requires some clever hacks to get working, but as a result, it's several times slower than HPC.js.

In WebGPU, commands are executed asynchronously, meaning that even when we perform [`GPUKernel.run`](../api/gpukernel/run), the driver might decide to wait a few milliseconds before actually running it. Essentially, instead of telling WebGPU to do something, we "ask" it to do something, and it'll do it whenever it wants, in the order we request.

## Command Queues

All this means that just because a command has finished on the CPU, we can't guarantee that it's finished executing on the GPU. The driver keeps track of which commands still need to be executed using a _command queue_, and running a method on the CPU simply adds to this queue.

This distinction is extremely important for methods like [`writeBuffer`](../api/gpu/writebuffer). Even though this method completes almsot instantly, it may actually take a while (up to tens of milliseconds) to complete on the GPU! This delay means that other commands, like `GPUKernel.run`, will be forced to wait until the previous command completes.

## Async Methods

Some methods, like [`readBuffer`](../api/gpu/readbuffer.md) are marked `async`. Our reasoning for doing this is twofold: first, these methods can take so long to execute that it wouldn't make sense to block CPU execution until they complete. Second, if you want to wait for them to finish, you can `await` them.

`readBuffer`, for example, needs to write to a `Float32Array` on the CPU, and thus it can't simply be added to the command queue. For large buffers, it doesn't make sense to regularly block execution with this command, so instead we handle it using the `Promise` API.
