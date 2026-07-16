# Forensics: Lateral Movement: groundtruth

For Q1, I searched the Security log for successful logons. The suspicious account was `DTEclass\svc_backup`, and its SID was shown in Event ID 4624.
```text
SSS{S-1-5-21-2621840962-3112170429-1249113562-2614}
```

For Q2, I checked the Terminal Services Remote Connection Manager log. Event ID 1149 showed a successful RDP authentication for `svc_backup` at:
```text
2026-07-16 10:27:33
```

For Q3, I searched Event ID 4688 for process command lines. I found a `net use` command containing the password in plaintext: `net use \\192.168.181.10\NETLOGON /user:lab\svc_backup Svc_back_up11111`
The password flag was: `SSS{Svc_back_up11111}`
