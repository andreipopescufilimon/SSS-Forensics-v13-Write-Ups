# nikola

**Q1. What credentials does the attacker use?**

I used this command: 
```
tshark -r nikola.pcap -Y 'ftp.request.command == "USER" || ftp.request.command == "PASS"' \
```

Receiving:
```
USER    edunis@corwineagles.com
PASS    cCycU=91vup7
```

---

**Q2. Who is the victim?**

I used:
```
tshark -r nikola.pcap -Y ftp -T fields \
-e frame.number -e ip.src -e ip.dst \
-e ftp.request.command -e ftp.request.arg \
-e ftp.response.code -e ftp.response.arg
```

and got:
```
307     162.241.123.75  10.2.3.101                      227     Entering Passive Mode (162,241,123,75,183,189)
312     10.2.3.101      162.241.123.75  STOR    PW_tyler-DESKTOP-W7F98GR_2026_02_03_16_13_59.html
```

---

**Q3. What files were exfiltrated?**

I used:
```
tshark -r nikola.pcap \
  -Y 'ftp.request.command == "STOR"' \
  -T fields -e ftp.request.arg
```

and got:
```
PW_tyler-DESKTOP-W7F98GR_2026_02_03_16_13_59.html
Contacts_Thunderbird.txt_tyler-DESKTOP-W7F98GR_2026_02_03_16_14_02.txt
```

---

**Q4. Whoa, is that an OAuth token?**

I used:
```
tshark -r nikola.pcap \
  -Y 'ip.src == 10.2.3.101 && tcp.dstport == 47037 && tcp.len > 0' \
  -T fields -e tcp.payload |
tr -d '\n' |
xxd -r -p > stolen-passwords.html

grep -o -i 'Host: oauth://[^<]*\|Username: [^<]*\|Password: [^<]*' stolen-passwords.html
```

and got:
```
Host: oauth://accounts.google.com
Username: jane.roberts7942@gmail.com
Password: MS8vMGZuM0lROG56ZTB0OUNnWUlBUkFBR0E4U053Ri1MOUlyT0pkU0xSaDJwSDJHR0dTUHNzVmJDMDR5NUlySW9hajl4bG5IRmhTVnluVFljZjZtMDgxeF9xRnRtVko1OHZHYTVVRQE=
Username: jane.roberts7942@gmail.com
Password: 9!2*D4b~p]h2Q7;"K5xBgZ
Username: jane.roberts7942@gmail.com
Password: 9!2*D4b~p]h2Q7;"K5xBgZ
```

so the username was `jane.roberts7942@gmail.com` and the token was `1//0fn3IQ8nze0t9CgYIARAAGA8SNwF-L9IrOJdSLRh2pH2GGGSPssVbC04y5IrIoaj9xlnHFhSVynTYcf6m081x_qFtmVJ58vGa5UE` after being decoded from base 64.

---

**Q5. How many contacts did they steal?**

I used:
```
tshark -r nikola.pcap \
  -Y 'ip.src == 10.2.3.101 && tcp.dstport == 31521 && tcp.len > 0' \
  -T fields -e tcp.payload |
tr -d '\n' |
xxd -r -p |
grep -c '@'
```

and got:
```
25
```

---

**Q6. What stealer did the attacker use?**

I listed the files uploaded by the malware over FTP and got:
```
PW_tyler-DESKTOP-W7F98GR_2026_02_03_16_13_59.html
Contacts_Thunderbird.txt_tyler-DESKTOP-W7F98GR_2026_02_03_16_14_02.txt
```

The PW_*.html password file and Contacts_Thunderbird.txt_* filename pattern are associated with Agent Tesla FTP exfiltration.

SSS{AgentTesla}
