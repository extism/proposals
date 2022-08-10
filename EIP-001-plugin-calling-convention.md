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

- `input_offset() -> i32`
  - Returns the start offset of the input data
- `input_length() -> i32`
  - Returns the length of the input data
- `output_set(offset: i32, length: i32)`
  - Sets the offset/length for output data
- `alloc(length: i32) -> i32`
  - Allocates `length` bytes and returns the offset, this can be used to allocate the output data
  
On the SDK side we provide the following API:

- `output_length(plugin_id: i32) -> i32`
  - Returns the length of the output buffer, this is used so the caller can allocate a big enough buffer to copy
    the data
- `output_get(plugin_id: i32, buffer: *mut void, length: i32)`
  - Copies the output buffer into `buffer`