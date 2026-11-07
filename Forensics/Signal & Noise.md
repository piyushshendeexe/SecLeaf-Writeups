# Signal and Noise

**Category:** Network Forensics

## Analysis

Parsed the raw IPv4 PCAP using Python.

- Total packets: `2211`
- Victim: `10.0.88.10`
- Protocols: ICMP (800), UDP (731), TCP (680)

ICMP traffic consisted of 800 small packets from 50 sources. Its comparatively low volume made it a decoy/noise vector.

## Attack Vectors

**1. TCP SYN Flood**

- Total SYN packets to victim: `560`
- From `172.16.80.x`: `500`
- Distinct flood sources: `41`

**2. DNS Reflection / Amplification**

- Spoofed DNS queries: `350`
- DNS responses to victim: `350`
- Reflectors: `4`
- Query size: `68 bytes`
- Response size: `714 bytes`
- Query: `big-record.example.org`
- QTYPE: `ANY (255)`

Amplification factor:

\[
714 / 68 = 10.5\times
\]

Reflector distribution:

| Reflector | Responses |
|---|---:|
| `198.18.0.11` | 86 |
| `198.18.0.22` | 96 |
| `198.18.0.33` | 84 |
| `198.18.0.44` | 84 |

## Pivot Identification

IP `172.16.80.99` appeared in both attack vectors:

- Sent 29 TCP SYN packets to the victim.
- Sent 31 DNS queries to the four reflectors.
- Queried `big-record.example.org` as part of the DNS amplification activity.

This identifies `172.16.80.99` as the shared orchestrator/pivot IP.

## Final Findings

| Field | Result |
|---|---|
| Victim | `10.0.88.10` |
| Main vectors | TCP SYN flood + DNS amplification |
| Decoy/noise | ICMP echo traffic |
| DNS reflectors | 4 |
| Amplification | `10.5×` |
| Pivot IP | `172.16.80.99` |

## Flag

`FLOWTRIQ{decoy_icmp_pivot_172.16.80.99}`
