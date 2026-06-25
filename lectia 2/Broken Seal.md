# Broken Seal

I started by checking the provided SHA256 hashes against the extracted artifacts:

```bash
sha256sum -c hashes.txt | grep FAILED
```

The output showed one failed file:

```txt
C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker Encrypt All Drives: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```

This meant that one file did not match the expected hash. The suspicious file was a Windows Scheduled Task related to BitLocker.

Next, I checked the file type:
```bash
file C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker\ Encrypt\ All\ Drives
```

The result showed that it was an XML task file encoded in UTF-16:
```txt
C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker Encrypt All Drives: XML 1.0 document, Little-endian UTF-16 Unicode text, with CRLF line terminators
```

Since the file was UTF-16, I converted it to UTF-8 so it could be read properly:
```bash
iconv -f UTF-16 -t UTF-8 C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker\ Encrypt\ All\ Drives
```

Inside the XML, I found an interesting modified action:

```xml
</Actions>
<Actions HashIntegrity="failed">
        <ClassId>%%%M:dEx1HcD;va5d&;`@O</ClassId>
</Actions>
```

The `HashIntegrity="failed"` attribute confirmed that this part of the file was suspicious. The value inside `ClassId` looked encoded: `%%%M:dEx1HcD;va5d&;`@O`

This produced: `TTT|i5tI`w4sjG2d5Uj1o~`

This decoded the real flag: `SSS{h4sH_v3riF1c4Ti0n}`
