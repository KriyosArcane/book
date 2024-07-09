# Extism Plug-in

PumpBin uses [Extism](https://github.com/extism/extism) to implement the plugin system.

> Note: One of the primary use cases for Extism is building extensible software & plugins. You want to be able to execute arbitrary, untrusted code from your users? Extism makes this safe and practical to do.

PumpBin Extism Plug-in input and output are both byte streams, and use json serialization. The plug-in needs to deserialize the input byte stream into json and serialize the json output into a byte stream.
An example of using [serde_json](https://crates.io/crates/serde_json) in rust is as follows:

```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct Input {
    ...
}

#[derive(Debug, Serialize, Deserialize)]
pub struct Output {
    ...
}

#[plugin_fn]
pub fn extism_plugin(input: Vec<u8>) -> FnResult<Vec<u8>> {
    let input = serde_json::from_slice::<Input>(input.as_slice())?;
    ...
    let output = Output { ... };
    Ok(serde_json::to_vec(&output)?)
}
```

<div class="warning">

Common errors

If there is any code in the plug-in that involves syscalls, you need to compile it with the `wasi` module enabled. For example, generating random numbers, network access, etc.
(It is recommended to compile it with the `wasi` module enabled directly to avoid errors. In rust, it is the `wasm32-wasi` target)

</div>

Extism has a [detailed tutorial](https://extism.org/docs/quickstart/plugin-quickstart) on developing plug-ins in various languages. Below is the API documentation for the PumpBin Extism plug-in.

- [encrypt_shellcode](api/encrypt_shellcode.md)
- [format_encrypted_shellcode](api/format_encrypted_shellcode.md)
- [format_url_remote](api/format_url_remote.md)
- [upload_final_shellcode_remote](api/upload_final_shellcode_remote.md)

The plugin that ends with `remote` is a plugin of the `Remote` type.
