# switcheroo

## Q1: What file was staged for the privesc?

The hint mentions a privilege-escalation technique discussed in the Linux section and a specific UAC artifact. This points to SUID binaries.
After extracting the archive: `tar -xzf switcheroo.tar.gz
I inspected the UAC SUID artifact: `cat system/suid.txt`
A suspicious entry appeared under `/tmp`: `/tmp/.font-unix/fc-cache`

## Q2: What time did the attacker stage the privesc?

I searched the UAC bodyfile for the suspicious path: `grep '/tmp/.font-unix/fc-cache' bodyfile/bodyfile.txt`
The entry contained the Unix timestamp: `1781201328`
UAC bodyfile timestamps are Unix epochs, so I converted it directly as UTC: `date -u -d @1781201328 '+%Y-%m-%d %H:%M:%S'`
Output: `2026-06-11 18:08:48`
