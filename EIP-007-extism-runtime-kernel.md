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

## Hosts

To access the plugin input and output, it is easiest to create functions like `Plugin.set_input`, `Plugin.get_output` and `Plugin.get_error`
to handle the Extism memory access. On the host side a `Plugin.call` function might look something like this in pseudocode:

```
fn Plugin.call(self, name: string, input: []u8) -> []u8 {
  self.reset(); // call to extism_reset
  self.set_input(input); // set extism input
  let f = self.get_function(name); // get function from module
  let rc = f.call(); // call the WASM function
  if rc != 0 { // handle errors
    throw(self.get_error());
  }
  return self.get_output() // return output
}
```

The additional functionality not provided by the kernel will have to be implemented for each host. The following functions should
be defined as host functions in order to create a full-featured Extism host:

### Config

- `extism_config_get(I64) -> I64`: Get configuration value
  - Get config value, takes an offset to the name in memory, returns an offset to the config value in memory, or 0 if not found 

### Var

- `extism_var_get(I64) -> I64`
  - Get var, takes an offset to the name in memory, returns an offset to the var value in memory, or 0 if not found 
- `extism_var_set(I64, I64)`: 
  - Set var, takes an offset to the name in memory and an offset the the value

### HTTP

- `extism_http_request(I64, I64) -> I64`
  - Make HTTP request, takes an offset to a JSON object compatible with `extism_manifst::HttpRequest` and an offset to the request body
    in memory (or 0 if there is no request body)
  - Returns an offset to the response body in memory
- `extism_http_status_code() -> I32`
  - Get status code of last HTTP request

### Log

These functions all take an offset to a log message in memory and return nothing:

- `extism_log_warn(I64)`
- `extism_log_info(I64)`
- `extism_log_debug(I64)`
- `extism_log_error(I64)`

