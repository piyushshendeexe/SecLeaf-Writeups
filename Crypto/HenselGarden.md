# Hensel Garden

**Category:** Cryptography

### Challenge Analysis

The challenge provides two modular square observations:

\[
(s+87)^2 \bmod 2^{72}
\]

\[
(s+143)^2 \bmod 3^{51}
\]

The seed is 128-bit, and the encryption key is derived using SHA-256.

### Approach

1. Find square roots modulo \(2^{72}\) and \(3^{51}\) using Hensel lifting.
2. Combine the roots using CRT (Chinese Remainder Theorem).
3. Keep candidates within the 128-bit seed range.
4. Derive the AES key and verify candidates using the AES-GCM authentication tag.

### Solve

```bash
python3 solve.py
```

Output:

```text
4 2 1 1
282779946134366997745983790016833981964
b'SecLeaf{r00ts_gr0w_through_pr1me_p0w3rs}'
```

### Recovered Seed

```text
282779946134366997745983790016833981964
```

### Flag

```text
SecLeaf{r00ts_gr0w_through_pr1me_p0w3rs}
```

### Key Techniques

- Hensel Lifting
- Modular Square Roots
- Chinese Remainder Theorem (CRT)
- AES-GCM Verification
