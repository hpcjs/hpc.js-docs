---
sidebar_position: 4
---

# Bouncing Balls

This tutorial will introduce you to the basics of physics simulation, utilizing `vecN` buffers and `Math.random()` on the GPU. By the end, we'll have a simulation of hundreds, if not thousands, of colorful balls bouncing arond the screen!

![bouncing balls](../../static/img/bouncing-balls.png 'Bouncing Balls')

## The Plan

### Physics Simulation Basics

Each ball on the screen has two properties we care about: position and velocity, also known as `x` and `v`. As we learned in high school, velocity is the time-derivative of position; in other words, `v = Δx/Δt`. Clearing the fraction, we see `Δx = vΔt`, meaning that to get the change in position within a single frame, we need to multiply a ball's velocity by the time passed within a frame.

Velocity works the same way. Acceleration is the time-derivative of velocity, so `a = Δv/Δt` => `Δv = aΔt`. Luckily, we already know the acceleration on earth: `9.81 m/s^2` downwards! This allows us to update the velocity of the balls at each frame, and we can then use this new velocity to update the position.

Finally, collisions. In our case, we'll just be colliding with the walls today, and all we need to do for that is check if the ball is closer to the wall than its radius: `position.x < ballRadius || position.x > screenSize - ballRadius`. If it is, then the ball must inside the wall, so we'll have to correct its position and flip the velocity away from the wall.

If any of this is confusing, don't worry! It'll all become clear when we implement it in code.

### The Program

In order to keep track of the balls' positions and velocities, we'll have two `vec2` buffers. Additionally, we'll have a `vec3` buffer storing the color of each ball. Each ball will have the same radius, so we'll keep that in a uniform variable. We'll also store gravity as a uniform `vec2`, and `dt` will be in there too, so we can control how fast the simulation runs. To control the number of balls, we'll make a `NUM_BALLS` constant that we can easily adjust.

:::info

We've gone ahead and created a Next.js scaffolding project so you can get going immediately. We'll be doing everything in the `async run()` function inside the `useEffect` hook. Once you've got that up and running, continue on to the tutorial!

:::

## Initialization

There are two steps to our initialization. First we need to setup the `GPUInterface`, and afterwards we need to set the balls' positions, velocities, and colors.

### Setting up the GPUInterface

We need to fetch the canvas from the DOM in order to supply it to the `GPUInterface`. We'll set `NUM_BALLS` to 300, the ball radius to 10, and `dt` to 1/10th of a second.

```ts
const NUM_BALLS = 300;

const canvas = document.querySelector('canvas')!;
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const gpu = new GPUInterface({
  buffers: [
    {
      name: 'positions',
      type: 'vec2',
      size: [NUM_BALLS],
    },
    {
      name: 'velocities',
      type: 'vec2',
      size: [NUM_BALLS],
    },
    {
      name: 'colors',
      type: 'vec3',
      size: [NUM_BALLS],
    },
  ],
  uniforms: {
    gravity: vec2(0, -9.81),
    radius: 10,
    dt: 0.1,
  },
  canvas,
});

await gpu.initialize();
```

### Ball Initialization

If you wanted, you could create arrays with the balls' initial configurations and pass them as `initialData` parameters to the `GPUInterface`, but we're GPU programmers, so let's parallelize this! We'll be creating a kernel called `init` that runs once for each ball, only at the start of the program.

To set the ball's position, we select a random spot somewhere on the canvas.

```ts
const init = await gpu.createKernel(inputs => {
  const id = inputs.threadId.x;
  const size = inputs.canvas.size;

  inputs.buffers.positions[id] = vec2(
    Math.random() * size.x,
    Math.random() * size.y
  );
});
```

For velocity we'll pick two random numbers between -50 and 50.

```ts
inputs.buffers.velocities[id] = vec2(
  Math.random() * 100 - 50,
  Math.random() * 100 - 50
);
```

And for color, we'll set each component as a random number between 0 and 255.

```ts
inputs.buffers.colors[id] = vec3(
  Math.random() * 255,
  Math.random() * 255,
  Math.random() * 255
);
```

That's our initialization kernel complete! To run it once for each ball, just write

```ts
init.run(NUM_BALLS);
```

## Main Kernel Code

With the initialization done, it's time to write the main loop code. It'll consist of three kernels: performing the physics simulation, clearing the screen, and drawing the balls respectively.

### Physics Simulation Kernel

Let's begin by fetching all the variables we'll need.

```ts
const step = await gpu.createKernel(inputs => {
  const id = inputs.threadId.x;

  const pos = inputs.buffers.positions[id];
  const vel = inputs.buffers.velocities[id];

  const gravity = inputs.uniforms.gravity;
  const radius = inputs.uniforms.radius;
  const dt = inputs.uniforms.dt;

  const size = inputs.canvas.size;
});
```

Next, we implement the [formulas](#physics-simulation-basics) discussed at the beginning of the article. Since `Δx = vΔt`, we can update `x` by saying `x += vΔt`, and update acceleration in a similar way.

```ts
let newPos = pos.plus(vel.times(dt)); // pos += vel * dt
let newVel = vel.plus(gravity.times(dt)); // vel += gravity * dt
```

Finally, the collision. We have two cases to consider, hitting the left wall and the right wall. If either of these happen, we correct the position and flip the velocity, simulating a "bounce" off the wall.

```ts
if (newPos.x < radius) {
  newPos.x = radius;
  newVel.x *= -1;
} else if (newPos.x > size.x - radius) {
  newPos.x = size.x - radius;
  newVel.x *= -1;
}
```

Followed by the same thing for the y coordinate:

```ts
if (newPos.y < radius) {
  newPos.y = radius;
  newVel.y *= -1;
} else if (newPos.y > size.y - radius) {
  newPos.y = size.y - radius;
  newVel.y *= -1;
}
```

After writing the updated position and velocity to their buffers, we're done!

```ts
inputs.buffers.positions[id] = newPos;
inputs.buffers.velocities[id] = newVel;
```

### Clearing the Screen

This kernel is really simple. We'll run it for each pixel, and it'll set the color to be a dark grey.

```ts
const clear = await gpu.createKernel(inputs => {
  const pixel = inputs.threadId.xy;
  inputs.canvas.setPixel(pixel, 32, 32, 32);
});
```

### Drawing the Balls

We'll spawn one thread per ball. Each thread will draw a "square", but ignore pixels outside the circle. To check if a pixel is inside the circle, we can use the distance formula: `x^2 + y^2 < r^2`.

Begin with fetching the relevant variables.

```ts
const draw = await gpu.createKernel(inputs => {
  const id = inputs.threadId.x;

  const pos = inputs.buffers.positions[id];
  const color = inputs.buffers.colors[id];

  const radius = inputs.uniforms.radius;
});
```

Then, all we need is to draw the circle. If we find that a pixel is inside the circle, we find its absolute screen position with `pos + vec2(x, y)` and set the pixel.

```ts
for (let x = -radius; x <= radius; x++) {
  for (let y = -radius; y <= radius; y++) {
    if (x * x + y * y < radius * radius) {
      inputs.canvas.setPixel(pos.plus(vec2(x, y)), color);
    }
  }
}
```

## Main Loop

Our main draw loop will proceed as follows: first we step the simulation, then we clear the screen and draw the balls, and finally update the canvas.

```ts
const loop = () => {
  step.run(NUM_BALLS);

  clear.run(canvas.width, canvas.height);
  draw.run(NUM_BALLS);
  gpu.updateCanvas();

  requestAnimationFrame(loop);
};

loop();
```

## Conclusion

That's it! If everything is working correctly, you should have a bunch of colorful balls bouncing around your screen.

To expand on the simulation, consider:

- Removing the call to `clear.run()` (looks cool, doesn't it?)
- Increasing the number of balls (try 10000 and beyond!)
- Changing the value of `dt` (how does the simulation change with large values?)

If you bring the number of balls past 65536, the number of visible balls won't increase. If you're curious on how to fix this, check out our article on [randomness](../api/kernels/randomness) (hint: increase the `numRandSeeds` parameter on the `GPUInterface` constructor).
