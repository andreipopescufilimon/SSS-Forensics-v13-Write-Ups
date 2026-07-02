# untitled

The fastest path was to identify the suspicious user context in memory, then recover the text that user had open in a process instead of doing a full filesystem reconstruction.

### 1. Identify the OS profile

First, confirm that the image is a Windows dump and get basic system metadata:

```bash
vol -q -f ftk.mem windows.info
```

This showed a 64-bit Windows 10/11-family system with snapshot time:

```text
2026-06-18 08:10:50+00:00
```

### 2. Look for suspicious user activity

List registry hives and running processes:

```bash
vol -q -f ftk.mem windows.registry.hivelist
vol -q -f ftk.mem windows.pslist
vol -q -f ftk.mem windows.cmdline
```

Two interesting artifacts stood out:

- a user hive for `C:\Users\systemid\ntuser.dat`
- `powershell.exe` spawning `notepad.exe` with `.\passwords.txt`

The key line was: `"C:\WINDOWS\system32\notepad.exe" .\passwords.txt`

That strongly suggested the attacker or rogue account had a plaintext credential or note file open at capture time.

### 3. Confirm the active user

Check the security context of the suspicious processes:
```bash
vol -q -f ftk.mem windows.getsids --pid 10084
vol -q -f ftk.mem windows.getsids --pid 3760
```

Both `powershell.exe` and `notepad.exe` were running as: `systemid`

So the next step was to recover the contents of the open Notepad buffer.

### 4. Dump the Notepad memory region

Search `notepad.exe` memory for the command-line artifact and dump the matching VAD:
```bash
vol -q -f ftk.mem windows.vadregexscan --pid 3760 --pattern '(?i)[\x20-\x7e]{0,200}passwords\.txt[\x20-\x7e]{0,1200}' --maxsize 4096
vol -q -o volout/vads -f ftk.mem windows.vadinfo --pid 3760 --address 0x232d9866df0 --dump --maxsize 16777216
```

Then extract readable UTF-16 strings from the dumped region:
```bash
strings -el volout/vads/pid.3760.vad.0x232d9860000-0x232d995ffff.dmp | nl -ba | sed -n '880,1020p'
```

This recovered the important plaintext directly from the Notepad buffer:
```text
The flag is RkZGe3N2WTNfZTNwYklyZUxfanZNNGVxZWx9
```

### 5. Decode the flag

The recovered string is Base64:
```bash
python3 - <<'PY'
import base64
s = 'RkZGe3N2WTNfZTNwYklyZUxfanZNNGVxZWx9'
print(base64.b64decode(s).decode())
PY
```

That yields: `FFF{svY3_e3pbIreL_jvM4eqel}`

Since the challenge flag format should begin with `SSS`, applying `ROT13` gives the final flag: `SSS{fiL3_r3coVerY_wiZ4rdry}`
