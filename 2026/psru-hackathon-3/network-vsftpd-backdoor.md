# Network — VSFTPD Backdoor Analysis

**CTF:** PSRU Cyber Hackathon #3  
**Category:** Network Forensics  
**Flag:** `/tmp/sCePH.b64`

## Description

> The SOC detected suspicious activity from an internal IP targeting an old FTP server running a vulnerable version of VSFTPD. Shortly after an FTP connection was made, an unauthorized outbound connection was observed from the server back to the attacker's machine. You are given a network packet capture taken during the incident. Analyze it to reconstruct the attack timeline and assess the damage.
>
> **Flag 2:** What is the name of the file used as a backdoor?

## Background

VSFTPD 2.3.4 has a famous backdoor: if a username ending in `:)` (smiley face) is sent during login, the server opens a root shell on **port 6200**. This was a supply chain attack introduced into the source code in 2011.

## Solution

### Step 1 — Filter FTP traffic

Opened `network03.pcapng` in Wireshark and applied the filter:

```
ftp
```

This showed all FTP control traffic. I looked for the telltale backdoor trigger — a `USER` command with a smiley face (`:)`) appended to the username.

### Step 2 — Find the backdoor shell

The VSFTPD 2.3.4 backdoor opens a shell on port 6200. I filtered for that:

```
tcp.port == 6200
```

This revealed packets on the backdoor port, confirming the exploit worked.

### Step 3 — Follow the TCP stream

Right-clicked any packet on port 6200 → **Follow → TCP Stream**.

This showed all commands the attacker ran on the server through the backdoor shell, including the file they dropped:

```
/tmp/sCePH.b64
```

## Flag

`/tmp/sCePH.b64`

## Takeaways

- VSFTPD 2.3.4 backdoor trigger: `USER <anything>:)` opens a root shell on port 6200
- Always check for well-known service backdoors when you see old software versions in a pcap
- Follow TCP Stream in Wireshark is the fastest way to reconstruct attacker activity after finding the right port
