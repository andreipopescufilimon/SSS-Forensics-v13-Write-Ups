# Droopy

The attackers tried to hide the source of the payload, but some traces were still left inside the collected artifacts. I started by searching through the extracted KAPE artifacts for suspicious references related to payload delivery.

First, I searched for obvious indicators: `grep -RinaE "http|https|ip|payload|download|powershell|curl|wget".`

Then I searched inside common artifact locations: `find . -type f | grep -iE "history|powershell|tasks|prefetch|event|startup|run|onedrive|webcache"`

In one of the python scripts I found `192.168.181.159:8888/payload`
