---
sidebar_position: 3
---

# Even-Odd Buffering

Even-odd buffering is a technique that can reduce unnecessary copies between buffers. It's a small speedup and typically won't be game-changing, but it's always good to keep in your back pocket.

## Introduction

In our [Heat Simulation Tutorial](../tutorials/heat-simulation), we avoided a race condition by writing our results to a temporary buffer and then copying the results back. This worked well, but it's a matter of fact that large, several-megabyte copies like this can be slow. Although we didn't mention it in the tutorial, there's an easy solution to this problem: even-odd buffering.

In our heat simulation program, we designated a "primary buffer" and a "temporary buffer". Instead, we could swap the roles of the buffers each frame: after computing the updated heat values, we could consider the temporary buffer as the new source of truth, and use the old "primary buffer" as our new "temporary buffer". On the next step, we'd swap the roles yet again, on and on.

## Implementation

To perform this technique, it's best to use a uniform variable to keep track of which buffer is the "primary buffer".

```ts
// probably have more stuff here
const gpu = new GPUInterface({
  uniforms: {
    step: 0
  }
})

...

let step = 0;
const loop = () => {
  kernel.run(1000);

  step++;
  gpu.setUniforms({ step });

  requestAnimationFrame(loop);
}

loop();
```

Additionally, since each buffer read/write is conditional on the current step, it may help to abstract those away into functions. Note that the order of the buffers is swapped in each functions, since they serve different purposes.

```ts
const kernel = await gpu.createKernel(inputs => {
  function read(index = types.number) {
    const step = inputs.uniforms.step;
    if (step % 2 === 0) {
      return inputs.buffers.data1[index];
    } else {
      return inputs.buffers.data2[index];
    }
  }

  function write(index = types.number, value = types.number) {
    const step = inputs.uniforms.step;
    if (step % 2 === 0) {
      inputs.buffers.data2[step] = data;
    } else {
      inputs.buffers.data1[step] = data;
    }
  }

  // other stuff
});
```

If you haven't noticed, we call it even-odd buffering because the behavior changes depending on whether the step is even or odd :)
