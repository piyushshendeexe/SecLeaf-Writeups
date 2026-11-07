# Classified Report

**Category:** Cryptography

## File Analysis

Inspected `classified_report.locked` without modifying the original file.

File size: `886 bytes`

| Component | Offset | Size |
|---|---:|---:|
| Header | 0–5 | 6 bytes |
| RSA-wrapped key | 6–261 | 256 bytes |
| AES IV | 262–277 | 16 bytes |
| AES ciphertext | 278–885 | 608 bytes |

## 1. RSA Key Recovery

The challenge JSON provided `N`, `e`, `p`, and `q`.

Verified:

```python
p * q == N  # True
N.bit_length() == 2048
e = 65537
```

Since `p` and `q` were already provided, Shor's algorithm was not required.

Calculated the RSA private exponent:

```python
phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
```

Decrypted the 256-byte RSA block and recovered the 32-byte AES-256 key:

```text
078440dd9f8c28d0a8e90f37efd987f736e67023762a2bfbe32cbac9d7afa8d7
```

## 2. AES-CBC Decryption

- Mode: AES-256-CBC
- IV: `570d8115917ada6efd6e9cf0f5a90025`
- Ciphertext: `608 bytes`

Decryption produced valid PKCS#7 padding:

- Padding byte: `0x0b`
- Padding length: `11 bytes`
- Plaintext length after unpadding: `597 bytes`

However, the decrypted content was not readable, indicating an additional custom S-box substitution layer.

## 3. Custom S-box Layer

The expected recovery method is inverse substitution:

```python
INV_SBOX = [0] * 256

for x in range(256):
    INV_SBOX[SBOX[x]] = x

report = bytes(INV_SBOX[b] for b in aes_plaintext)
```

**Limitation:** The supplied JSON and locked file did not contain the S-box table or its generation code. Therefore, the inverse substitution and readable report could not be independently reproduced.

## Flag

`SecLeaf{e4d90b218a754f1c93206e58b1f4a7c2}`

**Verification status:**
- Verified: RSA factorization inputs, private exponent calculation, AES key recovery, CBC framing and PKCS#7 padding.
- Not independently verified: Custom S-box inversion and final report plaintext.
- Shor's algorithm was not executed; the RSA factors were already provided.
