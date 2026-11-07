# Reflected Storm

**Category:** Network Forensics

## Analysis

Inspected `reflected-storm.pcap` and found 1,340 UDP packets. The traffic showed a clear anomaly:

- Victim: `10.0.55.200`
- Packets received: `600`
- Total received bytes: `608,400`
- Distinct source IPs: `5`

## Attack Identification

All 600 packets sent to the victim had:
- Source port: `53`
- Packet length: `1014 bytes`

The victim's outgoing packets had destination port `53` and length `75 bytes`, indicating DNS queries.

DNS payload analysis identified:

- Domain: `openresolver-test.example.com`
- Query type: `ANY (255)`
- Query count: `600`
- Response size: `1014 bytes`

This indicates a **DNS reflection and amplification attack** using five open resolvers.

## Amplification Factor

\[
\frac{1014}{75} \approx 13.52\times
\]

## Final Findings

| Field | Result |
|---|---|
| Victim | `10.0.55.200` |
| Reflectors | `5` |
| Protocol | UDP / DNS |
| Query type | ANY (255) |
| Amplification | `13.52×` |

**Note:** The two `tshark` commands were not executed; their expected reflector count of 5 is based on the Python analysis. No flag was provided in the supplied results.
