#  Module namespace for host functions

## Purpose

Determine the use of implicit or explicit external import module names. In wasm, a module declares the namespace or library name a function belongs
to when importing the function from the host. Currently, there is some non-standard use of module namespaces unless overriding it explicitly within
a module. 

## Solution

Use the `extism` namespace for all imports from language SDKs. We would provide this namespace from the runtime linker, and express the override in
each language SDK we distribute. See the support table included below.

## Considerations

#### Language support

When targeting wasm, a language would need to have built-in support to declare the namespace an external function comes from. 

Here's a tracking table for the languages we want to support and a reference to their library / module name override implementation:

Language support for module namespace declaration:

| Language | Known Support | Reference |
| ------------- |:-------------:| -----|
| C/C++ | ✅ | https://github.com/extism/hacking-examples/pull/3/files#diff-77099f0d7ab8883981627159d4e3628014a5bcfc19ea6faabc8524d0c67be06bR9 |
| Rust | ✅ |   https://rustwasm.github.io/docs/book/reference/js-ffi.html#from-the-rust-side |
| Grain | ✅ | https://github.com/grain-lang/grain/blob/205bd080e3006e8b6f3b56817eac61bbf8002275/stdlib/runtime/wasi.gr#L9-L12 |
| AssemblyScript | ✅ | https://www.assemblyscript.org/concepts.html#module-imports |
| Go | ✅ | https://github.com/tinygo-org/tinygo/issues/1120#issuecomment-631179445 |
| Swift | ✅ | https://book.swiftwasm.org/examples/importing-function.html#importing-a-function-from-host-environments |
| Zig | ❓ | [tbd](https://github.com/ziglang/zig/blob/e0178890ba5ad76fdf5ba955f479ccf6f05a3d49/lib/std/builtin.zig#L673-L678) |
| Kotlin | ❓ | https://github.com/JetBrains/kotlin/tree/ea836fd46a1fef07d77c96f9d7e8d7807f793453/libraries/stdlib/wasm |

#### Default behavior

It does not appear that _all_ languages use the same default namespace, should one not be specified.

E.g. Rust uses `env`, AssemblyScript uses `import`.

#### Clarity / Explicitness

There are benefits to using an explicit namespace in this context, such as the ability to immediately diagnose that a function is expected to
come from our runtime (consider using the `extism` module namespace). This is useful when debugging a module and seeing clearly that there are
functions used from Extism. Or if there are other functions expected, which are not within the `extism` namespace, it becomes easier to pinpoint
the issue and propose a fix.

Additionally, we can avoid symbol collisions or shadowing if there is ever support for module linking or something like the component model. Ideally,
we don't restrict the use of or conflict with other namespaced functions.
