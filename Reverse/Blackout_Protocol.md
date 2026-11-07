# Blackroot Archive

**Category:** Network Forensics  
**Event:** SecLeaf CTF  
**Team:** HACKBYTES

## 1. PCAP Analysis

Identified the provided capture:

```bash
file 1793449533219_blackroot_archive.pcap
```

```text
PCAP capture file, microsecond ts (little-endian)
version 2.4 (Ethernet)
```

The capture contained 119 packets (0–118). Scapy was used to inspect packets and extract raw payloads.

## 2. Network Traffic Investigation

Identified the following communication timeline:

| Packets | Protocol / Port | Activity |
|---|---|---|
| 0–3 | ARP | Internal network discovery |
| 4–23 | DNS | Internal and external DNS queries |
| 24–32 | HTTP | Ledger manifest download |
| 33–47 | UDP 9000 | Decoy metric traffic |
| 48–65 | SMTP 25 | Archive recovery key |
| 66–97 | TCP 4700–4703 | Decoy service beacons |
| 98–118 | TCP 4711 | Actual data exfiltration |

Victim workstation: `10.88.12.15`

## 3. HTTP Manifest Extraction

Inspected HTTP traffic between:

```text
10.88.12.15:52100 → 10.88.12.80:80
```

The requested resource was:

```http
GET /exports/ledger-sync.manifest HTTP/1.1
Host: docs.blackroot.internal
```

The response contained a Base64-encoded manifest split across three packets (28, 29 and 30).

After reassembling and decoding:

```text
BLACKROOT_ARCHIVE_MANIFEST
artifact=ledger-sync-2026
encoding=base64
key_source=mail:archive-notice
next_service=tcp/4711
```

This identified SMTP as the source of the recovery key and TCP/4711 as the next investigation target.

## 4. SMTP Credential Recovery

Inspected SMTP traffic between:

```text
10.88.12.22 → 10.88.12.25:25
```

Extracted email:

```text
Subject: Ledger archive synchronization

Archive route confirmed.
Recovery key: rootleaf
Use the key only with the ledger synchronization service.
```

<badge color="success">Recovery key found</badge>

```text
rootleaf
```

## 5. TCP/4711 Data Extraction

Inspected the external TCP connection:

```text
Destination: 198.51.100.144
Port: 4711
```

The custom protocol identified itself as:

```text
BLACKROOT-SYNC/1.0
session=ledger-sync-2026
mode=archive
```

The connection contained 10 records with hexadecimal `blob` values.

Example:

```text
record=01;blob=275d391e382237;state=ready
record=02;blob=0e28011b18092d;state=ready
...
record=10;blob=5f;state=ready
```

The fragments needed to be sorted by record number and concatenated before decryption.

## 6. Decryption

Created `solve.py` to automate the recovery process.

The script performs:

1. Extracts `blob` values from TCP/4711 payloads.
2. Sorts fragments by record number.
3. Converts hexadecimal strings into bytes.
4. Concatenates all fragments.
5. Applies repeating-key XOR using `rootleaf`.
6. Decodes the resulting Base64 string.

XOR operation:

```python
key = b"rootleaf"

decoded = bytes(
    b ^ key[i % len(key)]
    for i, b in enumerate(ciphertext)
)
```

Executed:

```bash
python3 solve.py
```

Output:

```text
xor out : b'U2VjTGVhZntleHBvcnRlZF9vYmplY3RzX3JldmVhbF90aGVfYXJjaGl2ZV9rZXl9'

b'SecLeaf{exported_objects_reveal_the_archive_key}'
```

## 7. Final Flag

```text
SecLeaf{exported_objects_reveal_the_archive_key}
```

## 8. Key Techniques

- PCAP Analysis using Scapy
- HTTP Stream Reconstruction
- SMTP Email Analysis
- TCP Payload Extraction
- Hexadecimal Decoding
- Repeating-Key XOR
- Base64 Decoding
- Network Traffic Correlation

## 9. IOCs

| Indicator | Value |
|---|---|
| Victim | `10.88.12.15` |
| Internal Document Server | `10.88.12.80` |
| SMTP Key Source | `10.88.12.22 → 10.88.12.25` |
| External Destination | `198.51.100.144` |
| Actual Exfiltration Port | TCP/4711 |
| Decoy Ports | TCP/4700–4703, UDP/9000 |
| Recovery Key | `rootleaf` |

**Note:** The PCAP establishes the network communication but does not identify which process opened TCP/4711. Host-side EDR logs would be required to confirm the responsible process.

**Team: HACKBYTES**
