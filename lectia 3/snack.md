# Snack

I started by checking the file type and readable strings:

```bash
file snack.bin
strings -a snack.bin
xxd snack.bin | head
```

The file was detected as raw data: `snack.bin: data`

Sleuth Kit could not identify a filesystem:
```bash
fsstat snack.bin
fls -r snack.bin
```

So I used `binwalk` to search for embedded data. I carved it out with `dd`: `dd if=snack.bin of=hidden.gz bs=1 skip=4046 status=none`
Then I decompressed it: `gzip -dc hidden.gz`

This revealed the flag: `SSS{sl4cK_sP4ce_h1dEs_s3creT5}`
