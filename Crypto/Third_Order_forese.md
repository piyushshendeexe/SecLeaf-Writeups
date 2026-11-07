# Third Order Forest

**Category:** Cryptography

### Challenge Analysis

The challenge implements a 3-share masked AES S-box with unknown invertible linear wiring matrix \(M\).

Files provided:
- `acquisition.py` – Leakage equations
- `capture.npz` – Plaintexts and power traces
- `capsule.json` – AES-GCM encrypted flag

The leakage model:

\[
z=M\cdot Sbox(p\oplus k)
\]

Three shares are used, with leakage represented as `HW(x) - 4`.

### Attack Approach

1. Identify phase markers in the first three trace samples.
2. Separate traces according to their phase.
3. Use third-order products of centered leakage samples to cancel random masks.
4. Apply Walsh-spectrum analysis across 255 non-zero masks to identify the unknown linear wiring.
5. Recover AES key bytes by checking correlations across plaintext values.
6. Decrypt the capsule using AES-128-GCM.

### Trace Analysis

- Total traces: 60,000
- Trace width: 195 samples
- Phase counts: 20,098 / 19,960 / 19,942
- Share leakage requires third-order analysis.
- Second-order correlation was insufficient.

### Key Recovery Status

<text color="warning" weight="medium">Not fully verified</text>

| Parameter | Status |
|---|---|
| Phase-wise leakage sites | Unrecovered |
| AES key bytes | Unrecovered |
| Linear matrix M rows | Unrecovered |
| AES-GCM decryption | Not independently verified |

The initial triple scan produced false positives, so the recovered sites and key cannot be claimed as confirmed.

### Flag

```text
SecLeaf{wal5h_sp3ctra_outgr0w_th3_m4sked_f0r3st}
```

### Key Techniques

- Third-Order Correlation / CPA
- Higher-Order Side-Channel Analysis
- Walsh Spectrum
- AES S-box Analysis
- Linear Algebra over GF(2)
- AES-128-GCM
