#  Name of proposal

Optional contexts

## Purpose

To simplify the simplest case of loading a single plugin from disk and calling a 
function.

## Solution

It is possible to accomplish this at the SDK level by embedding a unique `Context` 
inside each SDK's `Plugin` type equivalent. For users that don't need a `Context` 
to group plugins this relatively simple change enables new usage patterns without 
ripping out the existing interface.

For example, using the Python SDK the current `Context` interface is still available:

```python
with extism.Context() as ctx:
  plugin = ctx.plugin(manifest)

# Which is eqivalent to: 
ctx = extism.Context()
plugin = extism.Plugin(manifest, context=ctx)
```

But with this proposal a new interface is added that creates an implicit `Context` for 
`Plugin`s when one isn't provided. 

```python
plugin = extism.Plugin(manifest)
```

To implement this there is one small, but major breaking change to the SDKs. Instead of 
`context` being a required argument, it would become optional. This requires changing the
order of the arguments (which is an admittedly an annoying fix for Extism users).

```python
# Required
class Plugin:
  def __init__(self, context, manifest):
    self.context = context

# Default
class Plugin:
  def __init__(self, manifest, context=None):
    self.context = context or Context()
```

In Javascript this change is slightly more disruptive disruptive because the `context`
argument needs to be moved to the end of the argument list.

```javascript
# Required 
class Plugin {
  constructor(context, wasm, wasi = false, config = {}){
    ...
  }
}

# Optional
class Plugin {
  constructor(wasm, wasi = false, config = {}, context = null){
    if (context === null) context = new Context();
    ...
  }
}
```


Among languages with support for optional arguments this pattern is pretty consistent. 
Otherwise, a new function can be added.

```go
// NewPlugin creates a plugin in its own context
func NewPlugin(module io.Reader, functions []Function, wasi bool) (Plugin, error) {
	ctx := NewContext()
	return ctx.Plugin(module, functions, wasi)
}
```

In the case of Go, this ends up changing the API from

```go
ctx := extism.Context()
plugin, err := ctx.Plugin(wasm, functions, wasi)
```

to

```go
plugin, err := extism.NewPlugin(wasm, functions, wasi)
```


## Considerations

- Embedding the `Context` adds minor overhead
- An alternative to this would be to allow for `*mut Plugin` and `PluginIndex` to both
  exist in the C API (currently only `PluginIndex` is used to access a plugin within a 
  `Context`)
