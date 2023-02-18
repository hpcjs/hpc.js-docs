---
sidebar_position: 2
---

# GPU Compute Intro

## Motivation

If this is your first time being exposed to GPU compute, it may not be entirely clear exactly why it's been so revolutionary. An example may help:

> The RTX 3090, a top-of-the-line graphics card, has 10,496 cores available for compute. Accounting for memory access time and their speed relative to your CPU, let's estimate that your GPU has ~1000x the compute power of a single processor core. A job that would take **overnight** to run on your GPU would take **three years** to run on your CPU.

It's hard to put a comparison like that into perspective. Tech moves so fast that in three years, your work will be irrelevant. The question will change within three years, so why bother? GPU compute is the key to solving modern problems. It allows us to iterate quickly and rethink problems at 1000x speed.

## The Basics

Traditional CPUs execute instructions sequentially, meaning they'll only perform an operation after the previous one is complete. Years ago, IBM created the first multi-core processor that allowed two streams of instructions to be processed simultaneously. There's a limit to the number of cores that can exist on a single CPU (the best processors today may only have 16 cores), but GPUs are free to have over 10,000, meaning 10,000 instructions can be processed at the same time.

The applications of this power are overwhelming:

- Fluid simulations involve >100k particles all swimming past each other. The calculations we perform for each particle are roughly the same, so why force them to take turns? The GPU can process thousands of particles at once, making the impossible possible.
- Neural networks require millions of neurons to process inputs from millions of other neurons, but all they're doing is multiplying and adding. GPU compute is the reason deep learning exists today in its current form.

This may all be a bit abstract. Let's look at a more concrete example.

## Doubling an Array

One of the first things we learn in programming is to operate on arrays. We may want to take a list of numbers `[1, 2, 3, 4]` and double it into `[2, 4, 6, 8]`. In JavaScript, we would write

```ts
const mylist = [1, 2, 3, 4];
for (let i = 0; i < 4; i++) {
  mylist[i] *= 2;
}
```

This is ripe for GPU parallelization, as we're performing the same operation four times, and they're all independent of each other. Instead of having a single loop responsible for all numbers, why not have four GPU cores responsible for their own number?

```ts
const mylist = [1, 2, 3, 4];

// Call this with a different index on each core
function double(index: number) {
  mylist[index] *= 2;
}
```

It may seem like overkill, but once we begin dealing with 10 million numbers, the gains become apparent. As you begin creating GPU-based projects, the ideas will come. The GPU lends itself to certain kinds of work, and you'll think them up as soon as you become comfortable with the technology.
