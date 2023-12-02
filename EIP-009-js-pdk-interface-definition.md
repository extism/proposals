# JS-PDK Interface Defintion

## Purpose

We need to support host functions in the JS PDK but the current way we define
the interface, while working for exports, will not work for imports. Here is a rough
outline of how the current system works:

First we compile quickjs to wasm, we call this the "engine" or "core" module.
This module is a rust wasm app with quickjs embedded and has one custom function as an export `__invoke`.
`__invoke` takes an index (n), it looks at all the javascript exports in your main module, sorts them alphabetically,
then invokes the (nth) one.

The next step is to wizen and mutate this core module. The user passes in some javascript
which we eval and wizen. We need need to expose the exports in the export section. We do this
by evaling the js (in this compile step), reading the js module's exports, sorting them alphabetically,
and adding them to the exports table in order. The code for these functions (we call thunks) are also generated 
and just call `__invoke` with the export function id (where is sits in the order).

There are some downsides to doing this for exports, it's complicated and slow, but it also works fine.
This will not work for imports because we'd need to come up with a very wacky way to programatically extract
and type the imports from the user's js code. We can no longer magically assume we will know the wasm signature.
Exports are easy because all Extism exports have the same signature. Imports do not. Furthermore this process
of mutating a wasm module and adding some exports and functions works because it is a fairly additive process.
Importing on the other hand requires re-aligning a lot of other tables. And if i were to continue custom coding
that I'd be building a linker.

## Solution

What we need to start doing is have the programmer define for us their interface explicitly. They'll be able to
define the Exports and Imports in an IDL. We also need to use tooling to link the 
core module and the user code module so we don't fall down the trap of havign to build a linker.

Here is how I propose the new path will work. This may not be permanent but I think 
it will work for 1.0:

![js pdk pipeline](content/009-js-pdk-pipeline.png)

The programmer will define their interface in a typescript file `interface.d.ts`:

```typescript
// all extism exports have type () -> i32
interface Exports {
  myExport(): I32;
}

// this interface is optional and types are  defined by you
// to match your host functions
interface Imports {
  myHostFunc1(p: I64, q: I32): I64;
  myHostFunc2(p: I64): I64;
}
```

Step 1 of the CLI will be to generate a `shim.wasm` from this interface. The shim needs
to generate some thunk functions as well as import `__invoke` from the core module (which gets
linked later in the pipeline).

`shim.wasm` will be linked with `core.wasm` using [wasm-merge](https://github.com/WebAssembly/binaryen#wasm-merge)
which is kind of like a linker but a little bit higher level. I may also consider using `wasm-ld`
but this seems to work for me and is flexible.

Out of the merge comes the final module, but it's yet to be wizened. This is similar to the process
wer have now. We will run the module and eval the user's plugin js code in the egine. Then we freeze
and dump out to the final wasm after a few passes from wasm-opt.

TODO: add info on how imports and exports are wired up

## Considerations

### WIT

I considered using WIT for the interface but:

1. Typescript is more natural to JS programmers
2. I believe the binding generators come with all the ABI stuff along with it

We can always add WIT support too and support both.

### wasm-merge vs wasm-ld (or normal linker)

TODO
