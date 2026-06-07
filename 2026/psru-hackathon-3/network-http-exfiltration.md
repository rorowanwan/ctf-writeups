# Network — Data Exfiltration via HTTP

**CTF:** PSRU Cyber Hackathon #3  
**Category:** Network Forensics  
**Flag:** *(embedded in recovered image)*

## Description

> The SOC team detected abnormal traffic being sent continuously to an external domain. They suspect the attacker is using a basic command-line tool to gradually exfiltrate sensitive data through a common protocol to evade detection. Analyze the PCAP to understand the exfiltration technique and recover the stolen data.

## Solution

### Step 1 — Identify the exfiltration pattern

Opened `network04.pcapng` in Wireshark and filtered for suspicious HTTP GET requests:

```
http.request.method == "GET" && http.host contains "pinggy"
```

Clicked the first packet and expanded **Hypertext Transfer Protocol** in the Packet Details pane. The **Request URI** contained a `?filename=...` parameter — the attacker was chunking a file and encoding each chunk in the URL.

### Step 2 — Extract and decode the chunks

Used `tshark` to extract all the URI parameters, URL-decode them, reassemble the base64 chunks, and decode the final file:

```bash
tshark -r network-04.pcapng \
  -Y 'http.request.method == "GET"' \
  -T fields \
  -e http.request.uri \
  | grep "filename=" \
  | sed 's/.*filename=//' \
  | python3 -c "
import sys, urllib.parse, base64
chunks = [urllib.parse.unquote(l.strip()) for l in sys.stdin]
data = base64.b64decode(''.join(chunks) + '==')
open('output.jpg', 'wb').write(data)
print('Saved', len(data), 'bytes')
"
```

Alternatively, via Wireshark GUI: **File → Export Objects → HTTP** to save all transferred objects.

### Step 3 — Recover the flag

Opening `output.jpg` revealed an image with the flag written on it.

## Flag

*(image-based flag)*

## Takeaways

- Data exfiltration via URL query parameters is a common low-and-slow technique — easy to miss without traffic baselining
- Base64 chunked over HTTP GET requests is hard to detect at the protocol level since it looks like normal web traffic
- `tshark` with `-T fields` is much faster than manual Wireshark GUI work when reassembling multi-packet data
- Always check `http.request.uri` for unusual query parameters when investigating suspected exfiltration
