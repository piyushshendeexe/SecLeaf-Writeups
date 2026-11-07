# Eightfold Epitaph

**Category:** Cryptography  
**Event:** SecLeaf CTF  
**Team:** HACKBYTES

## 1. Reconnaissance

Extracted the challenge files:

```bash
unzip Eightfold_Epitaph.zip
file *
```

Files:
- `model.py` – Public arithmetic and cryptographic implementation.
- `data.json` – Calibration records, lane stamps, target stamp and encrypted capsule.

Important parameters:

```text
MOD  = 2^84
SIZE = 2^21
LANES = 8
```

The receipt is calculated using modular addition, not XOR:

\[
Receipt=\sum_{i=0}^{7}Label_i\pmod{2^{84}}
\]

## 2. Recovering the Affine Map

The stamp function was identified as:

\[
stamp(x)=ROL_{84}(m\cdot x+c,r)
\]

Tested all rotation values from 0 to 83 and recovered candidate affine maps using calibration records.

**Primary map – 15 inliers:**

```text
r = 35
m = 0x750e59bef1f0e6b385693
c = 0x67cd6060c7616d28d7c34
```

**Decoy map – 7 inliers:**

```text
r = 71
m = 0x78e4a9198f5ceca64eb75
c = 0x3b4224d942b574c008800
```

The remaining three records were identified as noise.

The majority map was selected as the correct transformation.

## 3. Inverting the Stamps

Applied the inverse transformation:

\[
unstamp(s)=
(ROR_{84}(s,35)-c)\cdot m^{-1}
\pmod{2^{84}}
\]

This recovered all eight lane seeds and the target:

```text
T = 0x1b59868b301e63f23871f
```

## 4. Wagner's Generalized Birthday Attack

The objective was to find one index from each of the eight lists satisfying:

\[
\sum_{i=0}^{7}Label_i\equiv T\pmod{2^{84}}
\]

Each list contained \(2^{21}\) possible labels.

Using Wagner's algorithm:

\[
\frac{84}{1+\log_2(8)}=21
\]

The problem was solved using a three-level merging strategy.

| Level | Operation | Matching Bits |
|---|---|---|
| 1 | Merge four pairs of lists | 21 |
| 2 | Merge resulting lists | 42 |
| 3 | Final matching | 84 |

Carry handling was implemented using separate 42-bit low and high limbs to correctly handle modular addition.

The resulting list sizes were:

```text
L1:
[2098083, 2097014, 2096744, 2098533]

L2:
[2099416, 2096046]

Final solutions:
1
```

Recovered indices:

```python
indices = [
    1924587,
    2031184,
    29597,
    785188,
    676753,
    560734,
    264902,
    1161509
]
```

Verified using the original model:

```python
model.receipt(indices, seeds) == T
```

Output:

```text
True
```

## 5. Capsule Decryption

Derived the receipt key:

```text
3d4d888000202e2db8c6f12732fbccb8
afb9992b442ce555089fc49a2d892cf7
```

The capsule contained a 12-byte nonce and 59-byte encrypted payload, including a 16-byte tag.

AES-GCM authentication attempts did not succeed with the tested AAD values.

The ciphertext was instead decrypted using AES-CTR with the GCM data counter starting at 2.

```python
Cipher(
    algorithms.AES(key),
    modes.CTR(nonce + (2).to_bytes(4, 'big'))
)
```

Recovered plaintext:

```text
SecLeaf{m0dular_c4rr13s_d0nt_r3st_1n_p34c3}
```

## 6. Final Flag

```text
SecLeaf{m0dular_c4rr13s_d0nt_r3st_1n_p34c3}
```

## 7. Key Techniques

- Modular Arithmetic
- Affine Transformation Recovery
- Calibration Poisoning Analysis
- Wagner's Generalized Birthday Attack
- CRT and Modular Inversion
- Carry Handling
- AES-CTR Decryption

**Note:** The AES authentication tag was not verified. The recovered plaintext was identified through the unique successful candidate producing readable flag-format output.

**Team: HACKBYTES**
