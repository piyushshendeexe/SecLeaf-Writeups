# Blackroot Archive

**Category:** Network Forensics

## Analysis

Inspected the 119-packet capture using Scapy.

- Finance workstation: `10.88.12.15`
- Docs server: `10.88.12.80`
- External server: `198.51.100.144`

The traffic revealed a multi-stage archive transfer.

## 1. Manifest Recovery

The finance workstation downloaded:

`/exports/ledger-sync.manifest`

from `docs.blackroot.internal`.

Reassembling the HTTP body and decoding Base64 revealed:

```text
BLACKROOT_ARCHIVE_MANIFEST
artifact=ledger-sync-2026
encoding=base64
key_source=mail:archive-notice
next_service=tcp/4711
```

This identified the SMTP email as the key source and TCP/4711 as the archive channel.

## 2. Recovery Key

An SMTP message from the ops workstation contained:

```text
Subject: Ledger archive synchronization

Recovery key: rootleaf
```

Key: `rootleaf`

## 3. Archive Extraction

TCP/4711 traffic contained numbered records with hexadecimal blobs.

Recovery process:

1. Extract `blob=` values from client-to-server payloads.
2. Sort records by record number.
3. Concatenate and decode the hexadecimal data.
4. Apply repeating-key XOR using `rootleaf`.
5. Base64-decode the result.

```python
key = b"rootleaf"
decoded = bytes(
    b ^ key[i % len(key)]
    for i, b in enumerate(ciphertext)
)
print(base64.b64decode(decoded))
```

Recovered output:

```text
SecLeaf{exported_objects_reveal_the_archive_key}
```

## IOCs

| Field | Value |
|---|---|
| Victim | `10.88.12.15` |
| Docs server | `10.88.12.80` |
| SMTP key source | `10.88.12.22` → `10.88.12.25` |
| Recovery key | `rootleaf` |
| External server | `198.51.100.144:4711` |
| Decoy traffic | TCP/4700–4703, UDP/9000 |
| Technique | Multi-stage transfer, repeating-key XOR, Base64 |

## Flag

`SecLeaf{exported_objects_reveal_the_archive_key}`

**Caveat:** The PCAP does not establish which process opened TCP/4711; host-side EDR or process logs are needed. The email's claim that the key is restricted to the ledger sync service is not evidence of technical enforcement. The other external ports were treated as decoys based on their observed payloads.
