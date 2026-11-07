# Suspicious Activity

**Category:** Network Forensics

## Reconnaissance

Analyzed `suspicious_activity.pcap` using Python and Scapy.

- Total packets: `1038`
- TCP packets: `768`
- DNS packets: `78` (39 queries + 39 responses)
- Victim: `192.168.1.20`
- DNS resolver: `8.8.8.8`

Most TCP traffic was HTTPS browsing noise. DNS queries were mostly normal, except repeated lookups for `evil.com`.

## DNS Exfiltration

Four suspicious queries contained Base64-like subdomain labels:

```text
U2VjTGVhZn.evil.com
tENDc0XzN4.evil.com
ZjFsX0RONX.evil.com
0=.evil.com
```

The labels were joined in capture order:

```text
U2VjTGVhZntENDc0XzN4ZjFsX0RONX0=
```

Base64 decoding recovered:

```text
SecLeaf{D474_3xf1l_DN5}
```

All four DNS queries received the same A-record response: `198.51.100.42`. The response carried no additional payload.

## IOCs

| Field | Value |
|---|---|
| Victim | `192.168.1.20` |
| Suspicious domain | `evil.com` |
| Answer IP | `198.51.100.42` |
| DNS resolver | `8.8.8.8` |
| Technique | DNS tunneling / Base64 exfiltration |
| MITRE ATT&CK | T1048.003, T1071.004 |

## Detection Indicators

- Repeated queries to one domain with changing, high-entropy subdomain labels.
- Base64-looking data embedded in DNS labels.
- Repeated queries receiving the same A-record answer.
- Workstation directly using public DNS resolver `8.8.8.8`.

## Flag

`SecLeaf{D474_3xf1l_DN5}`

**Caveat:** This conclusion is based on static PCAP analysis. HTTPS payloads were encrypted and not inspected, so other hidden payloads or activity cannot be ruled out. The recovered flag is supported by the successful Base64 decoding of the four DNS labels.
