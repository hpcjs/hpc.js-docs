# Functions

HPC.js lets you define custom functions. You can do whatever you want with them, but there are four minor caveats:

1. Functions must be defined using the `function` keyword, so no arrow functions!
2. Functions must be defined at the top level, before any other code
3. Nested function definitions are not allowed
4. Function parameter types must be annotated

That last one may be unfamiliar to some JavaScript developers. The reason we require it is simple: in normal HPC.js kernel code, we can easily infer the type of a given variable. In a custom function, however, the input variable types may not be so clear. We have no trouble deducing the function return type, so no need to specify it.

Because TypeScript type annotations are lost when transpiling to JavaScript, we created our own annotations, supporting `number`, `boolean`, `vec2`, `vec3`, and `vec4`. It's completely compatible with TypeScript, so no worries at all. To use it, simply import `types` from `hpc.js` and set your chosen type as the default value for your parameter:

```ts
import { types, vec3 } from 'hpc.js';

const kernel = await gpu.createKernel(inputs => {
  function double(v = types.vec3) {
    return v.times(2);
  }

  const x = vec3(1, 2, 3);
  const doubled = double(x);
});
```

The only issue is that because we're hijacking the default arguments syntax, functions will be marked as having all arguments optional. HPC.js doesn't currently support optional arguments, so don't be fooled: all parameters must be specified!
