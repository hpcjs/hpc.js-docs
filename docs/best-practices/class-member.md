---
sidebar_position: 1
---

# GPUInterface as a Class Member

In larger projects, it makes perfect sense to encapsulate your logic within a larger class, including a `GPUInterface` object as a private field. There are two common issues you may run into, both easily solved.

## Typescript Generics

If you try to type your class like this,

```ts
class MyClass {
  gpu: GPUInterface;

  constructor() {
    this.gpu = new GPUInterface({
      ...
    })
  }
}
```

you'll get an error complaining about generics. The reason is that behind the scenes, `GPUInterface` uses TypeScript generics to manage autocompletion. To avoid this, remove the type annotation, as it can be automatically deduced from the constructor assignment. If you like, you can include a comment.

```ts
class MyClass {
  gpu; // GPUInterface

  constructor() {
    this.gpu = new GPUInterface({
      ...
    })
  }
}
```

## Async Initialization

The other problem you may face is that you need to run `await gpu.initialize()`, but the constructor for your class can't be `async`. The solution is to add an `async initialize()` method to your class that calls the corresponding `GPUInterface` method.

```ts
class MyClass {
  gpu; // GPUInterface

  constructor() {
    this.gpu = new GPUInterface({
      ...
    });
  }

  async initialize() {
    await this.gpu.initialize();
  }
}
```

This "bubbles up" the initialization to the class's caller, which should be able to handle the `async` call properly.
