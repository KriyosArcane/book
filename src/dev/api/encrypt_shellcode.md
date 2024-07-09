# encrypt_shellcode

Encrypt the original shellcode.

## Function Name

encrypt_shellcode

## Input

```json
{
  "shellcode": []
}
```

The `shellcode` is a byte array and is the original shellcode.

## Output

```json
{
  "encrypted": [],
  "pass": [
    {
      "holder": [],
      "replace_by": []
    },
    {
      "holder": [],
      "replace_by": []
    }
  ]
}
```

`encrypted` is a byte array, which is the encrypted shellcode.

`pass` is an array of the following json structure, which is used to patch the encrypted password in the binary implant template (some encryption methods have multiple passwords, so it is an array):

```json
{
    "holder": [],
    "replace_by": []
}
```

The `holder` is a byte array and is a placeholder for the password in the decryption function of the binary implant template.
When sharing a plug-in, please inform how to correctly set the password in the decryption function.
For example, [aes256-gcm](https://github.com/pumpbin/plug-in/tree/main/encrypt_shellcode/aes256-gcm) in the [plug-in](https://github.com/pumpbin/plug-in) repository has the following README content:

> key: `$$KKKKKKKKKKKKKKKKKKKKKKKKKKKK$$`\
> nonce: `$$NNNNNNNN$$`

The above content shows that when using the `aes256-gcm` plug-in, the key in the AES256-GCM decryption function of the binary implant template needs to be set to `$$KKKKKKKKKKKKKKKKKKKKKKKKKKKK$$`, and the nonce to `$$NNNNNNNN$$`.

The `replace_by` is a byte array and is the encryption password. A random encryption password can be generated in the plug-in so that each generated final implant has a unique encryption password.
