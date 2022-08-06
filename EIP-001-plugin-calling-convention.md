#  Plugin calling convention

## Purpose

Determine how to pass input and output values between plugins and consumers using the types
available in WASM.

## Solution

We create 4 globals to store information about the location of the arguments and return value in memory:

- `in_offset`: is a pointer to the input data
- `in_length`: is the length of the input data
- `out_offset`: is a pointer to the output data
- `out_length`: is the length of the output data

In this example the following functions are available to the consumer:
  - `set_input(buffer, length)` is used to write input arguments
  - `get_output_length() -> u32` is used to get the length of the return value, this allows for the consumer to
    pre-allocate a buffer before calling `get_output`
  - `get_output(buffer)` is used to read the return value

When the consumer calls a function they (or a provided SDK) will call `set_input(buffer, length)` to copy data into
memory:

```
set_input(buffer, length):
  memcpy(memory[offset], buffer, length)
  store(in_offset, offset)
  store(in_length, length)
```

When the plugin returns it will store the return value in memory and update the globals:

```
return(value, length) =
  memcpy(memory[offset'], value, length)
  store(out_offset, offset')
  store(out_length, length)
```

`get_output_length` will return the contents of `out_length` so the consumer can allocate a buffer of the correct size,
then `get_output(buffer)` can be used to copy the data starting at `out_offset` into the provided buffer.

```
get_output_length() -> u32 =
  load(out_length)

get_output(buffer) =
  offset = load(out_offset)
  length = load(out_length)
  memcpy(buffer, offset as pointer, length)
```

## Considerations

The same thing could be accomplished with 1 global instead of 4. We can say data offset is always `1` if we can determine
that the input memory can be overwritten by the output memory. This would mean that we would just need a single global,
`data_length` to store the length of the current state.

Arguments:

```
set_input(buffer, length) =
  memcpy(memory[1], buffer, length)
  store(data_length, length)
```

Return:
```
return(value, length) =
  memcpy(memory[1], value, length)
  store(data_length, length)
```

Return value:

```
get_output_length() -> u32 =
  load(data_length)

get_output(buffer) =
  length = load(data_length)
  memcpy(buffer, memory[1], length)
```

This approach is a little more rigid because it required the input/output data to always be placed right at the start of the
memory block. It may also be the case that overwriting the input data isn't possible or makes things more complicated.
