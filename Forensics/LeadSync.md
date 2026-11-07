# LeafSync

**Category:** Network Forensics

## Analysis

Inspected `1793449295046_leafsync.pcap` using Scapy. The capture contained 55 packets.

- Victim: `10.20.30.15`
- Internal docs server: `10.20.30.80`
- Downloaded file: `/tools/leafsync-helper.sh`
- Host: `docs.internal.local`

The downloaded 52-byte shell script appeared to be a harmless stub.

## DNS Exfiltration

Shortly after the download, repeated DNS queries appeared for:

`<sequence>.<base64_chunk>.sync-cache.training.local`

All six queries returned `NXDOMAIN`.

Extracted Base64 chunks in sequence order:

```text
01 U2VjTGVh
02 ZntkbnNf
03 bGFiZWxz
04 X2hpZGVf
05 dGhlX3Ry
06 dXRofQ==
```

Joined and decoded:

```text
U2VjTGVhZntkbnNfbGFiZWxzX2hpZGVfdGhlX3RydXRofQ==
```

Decoded result:

```text
SecLeaf{dns_labels_hide_the_truth}
```

## IOCs

| Field | Value |
|---|---|
| Victim | `10.20.30.15` |
| Docs server | `10.20.30.80` |
| Download path | `/tools/leafsync-helper.sh` |
| Exfil domain | `sync-cache.training.local` |
| Technique | DNS tunneling using indexed Base64 chunks |

## Flag

`SecLeaf{dns_labels_hide_the_truth}`

**Caveat:** The PCAP establishes the DNS exfiltration pattern and timing, but cannot identify which host process generated the queries. Host-side audit or EDR logs are needed for process attribution.
