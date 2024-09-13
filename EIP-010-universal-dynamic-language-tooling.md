# Universal Dynamic Language Tooling

## Purpose

`extism-js` is a hyper focused and not very flexible tool.
It's simplicity has served us well, but my hope was that adding a new dynamic language
would give us an opportunity to build a universal tool for this
and establish some cross language conventions to make it easy for
extism users (or us) to add new (or customize extisting) runtimes.
Now that we're working on a python PDK, now might be a good time to do this.

### Creating new PDKs

Imagine if someone wants to add a ruby pdk. We should be able
to do that without creating new tooling to create and run these modules.

### Modifying official runtimes with your own native code

Another example which has come up, imagine if a user wants to customize our js runtime
with their own rust or c code, they should be able to fork our runtime
and re-use all our tooling and just publish their own runtime.wasm (essentially).

### Smaller plugins

The dynamic plugins will be orders of magnitude smaller as it will only be your code changes. You'll be able to cache the runtime in memory for the host app.

### Better development experience 

We could make some development mode tooling that will make feedback loops in development much faster and a better experience.

## Solution

There are still some details here I'm unsure of, but how i'd like for it to work is something like this.
What we're now calling `core.wasm` will effectively be a "runtime" (TODO: name pending as this is an overloaded term).
We could publish and version official runtimes. e.g. to start-off with we might have `quickjs` and `cpython`.
This should be publishable as plain wasm modules which both the tool, and the SDKs can fetch via url.
We might publish variants of these with different packages or purposes. e.g. we might
have a `runtime.development.wasm` with mode debug or development tools.
Or we could also publish variants like, a cpython that has numpy built in, etc.


The technical challenge mainly lies in defining the different modules we need and
what their interfaces are and doing that in a language agnostic way. I suspect
we can borrow from what we're already doing in our compiler toolchain with
wasm-merge, but saving the binding til runtime. See my [Draft PR](https://github.com/extism/python-pdk/pull/8)
in the python-pdk for a more concrete example.

### Separated Modules

Let me take a stab at what I think it would look like. First, what we are calling `core.wasm`
will instead be say `runtime.wasm`. This will contain a couple things:

1. The generic language runtime (e.g. quickjs, no application code)
2. The extism language bindings (e.g. in quickjs we include the rust-pdk and some binding code)
3. Common exports to invoke the runtime

> note: we may also have imports here for host functions? or we may need an additional import shim? more investigation needed.

We'll also have a `main.wasm`. This will be all the application code and will be the
stuff that actually changes from plugin to plugin. This should be small

### Module Interfaces

These need a consistent interface to be interchangeable and for common tooling
to be able to work with them (like wizen them, shim them, etc). The interface might look like this:

`runtime.wasm` will have many imports (probably extism, probably wasi, etc),
but will most importantly export 1 function: `__invoke`. This will do the invoke
trick established in the [js compiler](https://github.com/extism/proposals/blob/main/EIP-009-js-pdk-interface-definition.md).

I thinkt the above should work, though, we should consider if this is still necessary. What would be ideal is to have two exports like this:

* `__eval`
* `__evalByteCode`

Each of these could take a pointer to memory where the code is. Many runtimes support
evaluating both raw source as well as compiled bytecode specific to the vm.

`main.wasm` (final name of the file doesn't matter) would work as expected. It would import the interface from the runtime:

```
$ wasm-objdump main.wasm --section=Import -x

main.wasm:	file format wasm 0x1

Section Details:

Import[1]:
 - func[0] sig=0 <__invoke> <- runtime.__invoke
```

It would finally export the extism func:

```
wasm-objdump main.wasm --section=Export -x

main.wasm:	file format wasm 0x1

Section Details:

Export[1]:
 - func[1] <count_vowels> -> "count_vowels"
```

And we should be able to run by linking them up dynamically:

```
$ extism call main.wasm count_vowels --input="Hello World" --link core=./core.wasm  --wasi
{"count": 3}
```

### Tooling

From here we just need a way to manage all this in a single place. Perhaps
we can do it in the extism-cli. I don't think this can be written in go, but
we might be able to manage it as a "tool" to the go program. so it could be
a separate or linked program that's written in rust. But to the user, it should
ideally look like the extism cli.

I'll think a little bit more about what the experience should be. Ideally
a plug-in author should just be able to do `extism compile`
and not worry much about the underlying details.

## Considerations

### Wizer

I'm a little confused as to how this will work with wizer. I suspect the way my current python experiment is working,
it's wizening the core which is not what we want. [Javy](https://github.com/bytecodealliance/javy) seems to be able to do this
so there are probably some answers there. Furthermore, perhaps supporting an `evalByteCode` path might lessen the need for wizening?

### VMWare Workers Server

There is also some prior art to be studied with [vmware workers server](https://github.com/vmware-labs/wasm-workers-server).
They do something similar with the indepdent publishing of runtimes and dynamic linking. We may
be able to borrow some tricks or some of the user experience from there.

### Building Shims

How will shim building work exactly? We could perhaps have each runtime builder create some kind 
of program that can get the exports from the source code. But how will we distribute it? Could it be an extism
plug-in?
