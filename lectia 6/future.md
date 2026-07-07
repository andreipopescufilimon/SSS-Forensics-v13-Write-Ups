# Future

I started by filtering for HTTP requests inside the capture:

```bash
tshark -r future.pcap -Y "http.request" -T fields \
-e frame.number -e ip.src -e tcp.stream -e http.request.method \
-e http.host -e http.request.uri -e http.request.full_uri -e http.user_agent
```

This showed multiple HTTP GET requests from the victim machine `10.12.17.101` to `158.94.210.88`. The first request was:

```text
http://158.94.210.88/jaws
```

After that, the victim downloaded architecture-specific binaries from the same server:

```text
http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.x86
http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.mips
http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.mpsl
```


This matches the typical behavior of IoT malware loaders, where a small loader downloads different binaries depending on the target CPU architecture.


The challenge hint says:

```text
I did not expect a bot from the future.
```

The word “future” is the key hint. “Mirai” means “future” in Japanese. The payload structure also matches Mirai-style IoT malware, especially the use of architecture-specific binaries such as `mips`, `mpsl`, and `x86`.

Question 1: `SSS{http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.mips}`

Question 2: `SSS{mirai}`
