# upload_final_shellcode_remote

Upload the final shellcode to the remote server. (`Remote` Only)

The final shellcode is the encrypted and converted shellcode. If a plug-in is not set, it is returned as is. For example, if the format_encrypted_shellcode plug-in is not set,
the encrypted shellcode is returned as is.

## Function Name

upload_final_shellcode_remote

## Input

```json
{
  "final_shellcode": []
}
```

`final_shellcode` is a byte array, which is the final shellcode.

## Output

```json
{
  "url": ""
}
```

`url` is a string, which is the URL address after the upload, and PumpBin will automatically fill it into the shellcode URL input box.
