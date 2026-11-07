# Reflected Storm

**Category:** Network Forensics

### PCAP Analysis

Analyzed `reflected-storm.pcap` containing 1,340 UDP packets. The capture revealed a DNS amplification attack using spoofed source IP traffic.

### Investigation

**Victim IP:** `10.0.55.200`

**Identified 5 DNS reflectors:**
```text
198.51.100.22
203.0.113.5
192.0.2.17
198.51.100.10
203.0.113.88
```

**Attack characteristics:**

| Parameter | Value |
|---|---|
| Query Domain | `openresolver-test.example.com` |
| Query Type | ANY (255) |
| Outbound Packet Size | 75 bytes |
| Reflected Packet Size | 1014 bytes |
| IP Layer Amplification | 13.52x |
| DNS Payload Amplification | 20.98x |
| Packets Received by Victim | 600 |

### Verification

Count unique DNS reflectors:

```bash
tshark -r reflected-storm.pcap \
-Y "ip.dst == 10.0.55.200 && udp.srcport == 53" \
-T fields -e ip.src | sort -u | wc -l
```

Extract queried domain and query type:

```bash
tshark -r reflected-storm.pcap \
-Y "dns.flags.response == 0 && ip.src == 10.0.55.200" \
-T fields -e dns.qry.name -e dns.qry.type | head -n 1
```

### Conclusion

The PCAP confirms a DNS ANY-query reflection/amplification attack targeting `10.0.55.200`, using five DNS reflectors.

### Flag Extraction Values

```text
Victim IP       : 10.0.55.200
Reflectors      : 5
Domain          : openresolver-test.example.com
Query Type      : ANY (255)
Amplification   : 13.52x
```
