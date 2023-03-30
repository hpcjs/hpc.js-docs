---
sidebar_position: 3
---

# Heat Simulation

Of the tutorials presented so far, this one is the most involved. It uses almost all features offered by HPC.js, but if you're looking for a thorough introduction to the library, this is the place.

This tutorial will focus on creating a physically accurate heat diffusion simulation. We have a grid of heat cells, each represented by a "temperature" value ranging from 0 to 1, stored in a GPU buffer. Each frame, we step the simulation forward in time, react to user input (adding or removing heat), and update the canvas. The math required is minimal; even though advanced math is needed to solve the heat equation, we don't need any to actually use the algorithm.

At the end, you can expect something like this. As you can see, the heat bleeds across the canvas and melds together in a rather beautiful way. Of course, it's going to be interactive, so you'll be able to draw whatever you like.

![heat simulation](../../static/img/heat-simulation.png 'Heat Simulation')

## The Plan

### The Algorithm

The algorithm for diffusing heat is actually pretty simple. For each step, we have a given heat `h`, and we need to find `Δh`, the change in `h` for that step. The heat equation tells us that at each step, `Δh` is equal to the sum of all the differences between a cell's `h` and all of it's neighbors' `h` values. More explicitly:

`Δh[i][j] = (h[i + 1][j] - h[i][j])`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;` + (h[i - 1][j] - h[i][j])`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;` + (h[i][j + 1] - h[i][j])`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;` + (h[i][j - 1] - h[i][j])`

On each line, we add the difference between the neighbor and the current cell. If our neighbors have a higher temperature than ourself, then our `Δh` is positive and our temperature increases. If our neighbors are cooler than ourself, `Δh` is negative and our temperature decreases. If this doesn't make complete sense, don't worry; we'll see it in code!

:::note

You may notice that this doesn't exactly work when we're next to the wall, as we're missing a neighbor. In that case, we simply ignore the term including that neighbor.

:::

### The Program

Although the heat values are stored in a single buffer, we'll actually need two buffers. We do this because we need to avoid modifying heat values in-place: if we do that, the order of cell updates could change the resulting value. Instead, we'll write updates to a copy buffer, and then copy the result back to the original buffer all at once.

To add user interactivity, we'll make it so when the user left clicks, they add heat, and when they right click, they subtract heat. We'll also include a `radius` parameter that can be adjusted by scrolling.

:::info

We've gone ahead and created a Next.js scaffolding project so you can get going immediately. We'll be doing everything in the `async run()` function inside the `useEffect` hook. Once you've got that up and running, continue on to the tutorial!

:::

## Setting up the GPUInterface

We need to set up two buffers, one for heat and one for its copy. I've selected a size of `[900, 500]` since that's about the same ratio as a standard 1920x1080 screen, but you can choose anything (although we do recommend keeping it around the same aspect ratio as your monitor).

We're also going to use uniform variables to keep track of mouse position and the pressed button. We'll use `button = 0` for not clicking, `1` for left click, and `2` for right click.

```ts
const canvas = document.querySelector('canvas')!;
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const gpu = new GPUInterface({
  buffers: [
    {
      name: 'heat',
      size: [900, 500],
    },
    {
      name: 'heatcopy',
      size: [900, 500],
    },
  ],
  uniforms: {
    mouse: vec2(0, 0),
    button: 0,
    radius: 30,
  },
  canvas,
});

await gpu.initialize();
```

## Kernel Code

We're going to need two kernels: one to actuall run the the heat diffusion code, and one to draw the pixels to the screen. It's possible to do both of these in the same kernel, but it's easier to just implement them separately with negligible performance impact.

### Heat Diffusion

Let's get to writing our heat diffusion kernel! First, we'll set up our important variables.

```ts
const diffuse = await gpu.createKernel(inputs => {
  const mouse = inputs.uniforms.mouse;
  const button = inputs.uniforms.button;
  const radius = inputs.uniforms.radius;
  const cell = inputs.threadId.xy;
  const size = dim(inputs.buffers.heat);
});
```

Before doing any work, let's see if we can avoid it. If the cell is close to the mouse, and the user is clicking, we can skip heat computations and set the value directly.

```ts
if (mouse.dist(cell) < radius) {
  if (button === 1) {
    inputs.buffers.heatcopy[cell.x][cell.y] = 1;
    return;
  } else if (button === 2) {
    inputs.buffers.heatcopy[cell.x][cell.y] = 0;
    return;
  }
}
```

If this isn't the case, we'll have to calculate the heat delta like presented earlier. As noted, we need to check if our neighbors exist before we access them.

```ts
const current = inputs.buffers.heat[cell];
let delta = 0;

if (cell.x > 0) {
  delta += inputs.buffers.heat[cell.x - 1][cell.y] - current;
}

if (cell.x < size.x - 1) {
  delta += inputs.buffers.heat[cell.x + 1][cell.y] - current;
}

if (cell.y > 0) {
  delta += inputs.buffers.heat[cell.x][cell.y - 1] - current;
}

if (cell.y < size.y - 1) {
  delta += inputs.buffers.heat[cell.x][cell.y + 1] - current;
}
```

Finally, we can write our updated heat value to `heatcopy`. We need to multiply `delta` by a scaling factor to keep it from becoming too large. If the heat changes too rapidly, the simulation will become unstable.

```ts
const updated = current + delta * 0.25;
inputs.buffers.heatcopy[cell.x][cell.y] = updated;
```

And that's the diffusion kernel done! To copy the data back, we'll just use `GPUInterface.copyBuffer` later in the draw loop.

### Drawing

The drawing kernel is pretty simple. We'll run it once for each pixel, and for each pixel, we get the color of the cell it belongs to and set that as the brightness. Remember, the grid isn't necessarily the same resolution as the screen, so cells can take up multiple pixels.

First, we set up the variables:

```ts
const draw = await gpu.createKernel(inputs => {
  const bufferSize = dim(inputs.buffers.heat);
  const canvasSize = inputs.canvas.size;
  const pixel = inputs.threadId.xy;
});
```

Next, we need to rescale the pixel according to the size of the simulation in order to get the current cell. This can actually be accomplished in a single line thanks to `vec2` semantics.

```ts
const cell = pixel.times(bufferSize.div(canvasSize));
```

Then we just have to get the heat, scale to the range `0-255`, and set the color.

```ts
const heat = inputs.buffers.heat[cell.x][cell.y];
const color = vec3(heat * 255);

inputs.canvas.setPixel(pixel, color);
```

And the drawing is complete!

## User Input

User input is easily managed with JavaScript event handlers. First, we'll create a utility function to make the code a bit shorter. Notice how the `vec2` interface is the same on CPU and GPU!

```ts
const translate = (e: MouseEvent) => {
  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  const scale = vec2(900, 500).div(vec2(canvas.width, canvas.height));
  return vec2(x, y).times(scale);
};
```

We can pass this function a mouse location on the screen, and it'll automatically translate it relative to the canvas and scale it according to the grid size.

Next, we add the event listeners themselves.

```ts
canvas.addEventListener('mousemove', e => {
  const cell = translate(e);
  gpu.setUniforms({ mouse: cell });
});

canvas.addEventListener('mousedown', e => {
  const cell = translate(e);

  if (e.button === 0) {
    // left click
    gpu.setUniforms({ mouse: cell, button: 1 });
  } else if (e.button === 2) {
    // right click
    gpu.setUniforms({ mouse: cell, button: 2 });
  }
});

canvas.addEventListener('mouseup', () => {
  gpu.setUniforms({ button: 0 });
});

// we need this to prevent a context menu from appearing when we right click
canvas.addEventListener('contextmenu', e => {
  e.preventDefault();
});
```

## The Draw Loop

Bringing it all together, we assemble the draw loop as a reward for our hard work.

```ts
const loop = () => {
  diffuse.run(900, 500);
  gpu.copyBuffer('heatcopy', 'heat');

  draw.run(canvas.width, canvas.height);
  gpu.updateCanvas();

  requestAnimationFrame(loop);
};

loop();
```

## Conclusion

If everything has been done correctly, you should have a fully-functioning heat simulation! Here are some things we just trying:

- Increase the resolution from `[900, 500]` to match your canvas size
- Run `diffuse` and `gpu.copyBuffer` multiple times per loop to speed up diffusion
- Enable `useCpu` to see the speed difference between CPU and GPU

At this point, you should have a strong enough understanding of HPC.js to go on and create your own applications.
