# ospaperleak

After extracting the archive, I searched for `Amcache.hve`, since Amcache stores information about executed or seen binaries.
`find . -iname "Amcache.hve"`

Found: `./targets/C/Windows/AppCompat/Programs/Amcache.hve`

Because Windows registry hives store many strings as UTF-16 little endian, I used:
`strings -a -el "./targets/C/Windows/AppCompat/Programs/Amcache.hve" | grep -i -A6 -B6 "pe32_i386"`

The interesting entry was:
```00002a45d8a3bdffb38a59a98a8ca3f73394195b0595
pe32_i386
c:\users\test1\winpagedrip.exe
winpagedrip.exe
```

`pe32_i386` confirms it is a 32-bit executable, and winpagedrip.exe stands out as the suspicious dropped payload.
Amcache stores the SHA1 with a leading 0000, so: `00002a45d8a3bdffb38a59a98a8ca3f73394195b0595` becomes: `SSS{2a45d8a3bdffb38a59a98a8ca3f73394195b0595}`
