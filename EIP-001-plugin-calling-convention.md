#  Plugin calling convention

## Purpose

Determine how to pass input and output values between plugins and consumers using the types
available in WASM.

## Solution

Internally we use a struct to store information about the location of the arguments and return value in memory:

- `input_offset`: is a pointer to the input data
- `input_length`: is the length of the input data
- `output_offset`: is a pointer to the output data
- `output_length`: is the length of the output data

These values are not exposed outside of the core Rust implementation, exception through the following functions:

- `extism_input_offset() -> i64`
  - Returns the start offset of the input data
- `extism_input_length() -> i64`
  - Returns the length of the input data
- `extism_output_set(offset: i64, length: i64)`
  - Sets the offset/length for output data
- `extism_alloc(length: i64) -> i64`
  - Allocates `length` bytes and returns the offset, this can be used to allocate the output data
  
On the SDK side we provide the following API:

- `extism_output_length(plugin_id: i64) -> i64`
  - Returns the length of the output buffer, this is used so the caller can allocate a big enough buffer to copy
    the data
- `extism_output_get(plugin_id: i64, buffer: *mut void, length: i64)`
  - Copies the output buffer into `buffer`