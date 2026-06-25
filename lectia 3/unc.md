# unc

I extracted the archive and searched for download traces:
```bash
grep -RInaE "ZoneId|HostUrl|ReferrerUrl|https?://" .
```

A Windows Zone.Identifier entry showed:
```
ZoneId=3
HostUrl=https://unece.org/fileadmin/DAM/trade/agr/meetings/ge.01/document/2005_i05_apples_copacogeca.doc
```

ZoneId=3 means the file came from the internet, and HostUrl gives the original source.

Flag: `SSS{unece.org}`
