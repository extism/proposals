# Extism runtime kernel

## Purpose

To allow for Extism to be more portable across existing WebAssembly runtimes. Currently we only support Wasmtime+Rust
and compile a shared library that can be used from other languages. This would allow for us to build Extism support
on other runtimes (Wazero, Wasmedge, Wasmer, Wasm3, ...) in order to take advtange of language/runtime specific features
we're currently missing out on.

## Solution

In order to make the Extism runtime more portable, we can compile as much of the runtime as possible to WebAssembly and link it with
the user plugins. This should allow use to use the same memory management implementation across many different runtimes.

## Considerations

This will not break any existing plugins or Extism features (timeout, cancellation, memory limits) - there may be some additional
functions added to interact with the Extism kernel, but in general major API changes should be avoided.

Another thing to consider is how this should be organized - should the kernel live in it's own repo? Or along with the current
runtime? Where does it live after the SDKs are split into their own repos? It's likely that multiple SDKs will end up depending
on `extism-runtime.wasm`.
 
## Changes

### Input

Extism holds a pointer to the input and only copies it into memory using the `extism_input_load_*` functions. This
would require the input to be copied into the Extism memory all at once when a plugin is called. I haven't found this 
to cause any major performance hit. To make this work we will need to add an `extism_input_set` function which is called
from the host to specify a slice of memory that should be treated as input data (this is similar to how the `extism_output_set`
function works)

### Errors

Error tracking will also be moved into the kernel, it requires one new function:
- `extism_error_get`: To return the offset in memory that points to the current error (or 0 if it's not set)
