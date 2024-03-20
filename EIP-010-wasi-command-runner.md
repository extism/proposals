#  WASI Command Runner

## Purpose

Allow running WASI command modules with Extism. Make passing in and reading back data from a module easy when development with an Extism PDK isn't possible or desired. Continue expanding Extism to be a general purpose Web Assembly framework by adding a non-plugin oriented interface. Make running a command style module similar to running an Extism Plugin by providing a buffer in and buffers out interface. Such interface would be convenient for creating a library function around a command module for example.

## Solution

The interface would take in a buffer to use as STDIN data and return two buffers out, STDOUT and STDERR. As WASI command modules are setup to work with pipes for STDIN input data and STDOUT and STDERR output data, setting this up in various Web Assembly runtimes is non-trivial.

Environment variables and CLI args would also be passed as parameters.

For efficiency reasons, ideally the solution would only compile the module once, but instantiate the module on every call to run it as it's unsafe to run WASI module multiple times.

The persistent object holding the compiled module:
```rust
pub struct WASICommand {
    engine: wasmtime::Engine,
    module: wasmtime::Module,
}
```

The run function:
```rust
    pub fn run<'a, T: ToBytes<'a>, U: FromBytesOwned, V: FromBytesOwned>(
        &self,
        envs: &[(String, String)],
        args: &[String],
        stdin: T,
    ) -> Result<(U, V), Error> {
        let bytes = stdin.to_bytes()?;
        let stdin = ReadPipe::from(bytes.as_ref());
        let stdout = WritePipe::new_in_memory();
        let stderr = WritePipe::new_in_memory();
        let wasi_ctx = wasmtime_wasi::WasiCtxBuilder::new()
            .args(args)?
            .envs(envs)?
            .stdin(Box::new(stdin.clone()))
            .stdout(Box::new(stdout.clone()))
            .stderr(Box::new(stderr.clone()))
            .build();

        let mut store = wasmtime::Store::new(&self.engine, wasi_ctx);
        let mut linker = wasmtime::Linker::new(&self.engine);
        wasmtime_wasi::add_to_linker(&mut linker, |wasi| wasi)?;
        let instance = linker.instantiate(&mut store, &self.module)?;
        let function_name = "_start";
        let f = instance
            .get_func(&mut store, function_name)
            .expect("function exists");
        f.call(&mut store, &[], &mut []).unwrap();
        drop(store);
        let contents: Vec<u8> = stdout.try_into_inner().unwrap().into_inner();
        let stderr_contents: Vec<u8> = stderr.try_into_inner().unwrap().into_inner();
        let stdout_output = U::from_bytes_owned(&contents)?;
        let stderr_output = V::from_bytes_owned(&stderr_contents)?;
        Ok((stdout_output, stderr_output))
    }
}
```

https://github.com/extism/extism/tree/g4vi/wasi-command-runner

## Considerations

### Integration with existing Extism ecosystem manifest

Initially this wouldn't support the manifest at all. Supporting the/a manifest would require further consideration due to it having configuration items not applicable to non-plugin modules.