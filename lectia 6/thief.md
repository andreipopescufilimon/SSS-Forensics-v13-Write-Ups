# thief

## Q1. Who stole it?

First, I checked the IP conversations:
```bash
tshark -r thief.pcap -q -z conv,ip
```

The most suspicious conversation was between the victim `10.1.20.101` and `174.138.43.121`. This connection had the largest outbound transfer from the victim side, with about `5836 kB` sent to `174.138.43.121`. To confirm what the IP was connected to, I checked DNS:
```bash
tshark -r thief.pcap -Y "dns" -T fields \
-e frame.number -e ip.src -e ip.dst -e dns.qry.name -e dns.a -e dns.cname
```

The domain `whooptm.cyou` resolved to `174.138.43.121`. Then I checked HTTP requests:
```bash
tshark -r thief.pcap -Y "http.request" -T fields \
-e frame.number -e ip.src -e ip.dst -e http.request.method \
-e http.host -e http.request.uri
```

The victim was repeatedly contacting `whooptm.cyou` through `/api/set_agent`, including POST requests with browser fingerprinting data. So the attacker IP was: `SSS{174.138.43.121}`

## Q2. How many bytes did the victim send to the attacker?

To get the exact byte count, I calculated directional traffic:
```bash
tshark -r thief.pcap -Y "ip" -T fields -e ip.src -e ip.dst -e frame.len \
| awk '{bytes[$1" -> "$2]+=$3; frames[$1" -> "$2]++}
END {for (k in bytes) print bytes[k], frames[k], k}' \
| sort -nr | head -30
```

The largest outbound direction was: `5836953 4065 10.1.20.101 -> 174.138.43.121`. That means the victim sent `5836953` bytes to the attacker. Flag: `SSS{5836953}`

## Q3. What language does the attacker presumably use?

Next, I inspected the HTTP response body from `whooptm.cyou`. The returned JavaScript contained a Russian comment:
```js
// Преобразуем в URLSearchParams
```

This translates to something like “convert to URLSearchParams”. Because the attacker’s JavaScript contains a Russian comment, the attacker presumably uses Russian.

Flag: `SSS{Russian}`
