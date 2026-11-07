# Eightfold Epitaph

**Category:** Cryptography  
**Event:** SecLeaf CTF  
**Team:** HACKBYTES  
**Rank:** 50th

## Challenge Overview

The challenge, *Eightfold Funeral: Epitaph*, involves recovering a hidden flag from an encrypted capsule using modular arithmetic and a Generalized Birthday Attack.

The system consists of 8 lanes, each containing \(2^{21}\) virtual labels. The objective is to find one index from each lane such that the sum of their corresponding labels matches a target value modulo \(2^{84}\).

The challenge provides two files:

- `model.py` – Public arithmetic and cryptographic implementation.
- `data.json` – Calibration records, lane stamps, target stamp, and encrypted capsule.

**Key Hint:** Calibration records are not guaranteed to be honest, indicating the presence of poisoned or noisy records.

## 1. Reconnaissance

First, extracted the challenge archive and identified the provided files.

```bash
unzip Eightfold_Epitaph.zip
file *
```

Output:

```text
data.json: JSON data
model.py:  Python script
```

After analyzing `model.py`, the following parameters and functions were identified:

| Parameter | Value |
|---|---|
| Number of lanes | 8 |
| Virtual labels per lane | \(2^{21}\) |
| Modulus | \(2^{84}\) |
| Label size | 11 bytes |
| Calibration records | 25 |
| Capsule nonce | 12 bytes |
| Cipher | AES-CTR |

The stamp function was defined as:

\[
stamp(x)=ROL_{84}(m \cdot x+c,r)
\]

Each lane generates labels using SHA-256:

```python
label = sha256(
    b'Epitaph/v7|' + lane + seed + index
)[:11]
```

The final receipt is calculated as:

\[
Receipt=\sum_{i=0}^{7}Label_i \pmod{2^{84}}
\]

## 2. Recovering the Secret Affine Map

The first objective was to recover the hidden affine transformation parameters:

- Rotation value: \(r\)
- Multiplier: \(m\)
- Constant: \(c\)

For every possible rotation value from 0 to 83, the stamp values were rotated back using `ROR84`.

This converted the equation into:

\[
y=m\cdot x+c\pmod{2^{84}}
\]

Using differences between calibration records:

\[
m=\Delta y\cdot(\Delta x)^{-1}\pmod{2^{84}}
\]

The recovered candidates were:

**Primary affine map (15 inliers):**

```text
r = 35
m = 0x750e59bef1f0e6b385693
c = 0x67cd6060c7616d28d7c34
```

**Decoy affine map (7 inliers):**

```text
r = 71
m = 0x78e4a9198f5ceca64eb75
c = 0x3b4224d942b574c008800
```

The remaining 3 records were identified as noise.

The primary map was selected based on its higher number of consistent calibration records.

## 3. Inverting the Stamps

After recovering the correct affine parameters, the original values were extracted from the lane stamps and target stamp.

The inverse transformation is:

\[
unstamp(s)=
(ROR_{84}(s,35)-c)\cdot m^{-1}
\pmod{2^{84}}
\]

This recovered all 8 lane seeds and the target receipt:

```text
T = 0x1b59868b301e63f23871f
```

## 4. Generalized Birthday Attack (Wagner's Algorithm)

This was the main part of the challenge.

We needed to find eight indices, one from each lane, satisfying:

\[
\sum_{i=0}^{7}Label(seed_i,lane_i,index_i)
\equiv T \pmod{2^{84}}
\]

Each lane contained \(2^{21}\) possible labels.

A brute-force approach would be computationally expensive, so Wagner's Generalized Birthday Algorithm was used.

Since:

\[
\frac{84}{1+\log_2(8)}=\frac{84}{4}=21
\]

The available list size of \(2^{21}\) perfectly matched the required 21-bit collision size.

**Merge Strategy:**

| Level | Operation | Matching Bits |
|---|---|---|
| Level 1 | Merge (0,1), (2,3), (4,5), (6,7) | 21 bits |
| Level 2 | Merge the four resulting lists | 42 bits |
| Level 3 | Match the final two lists | 84 bits |

Carry handling was included during modular addition to ensure correct matching.

After completing the merge process, the following canonical solution was obtained:

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

The solution was verified using the original model:

```python
model.receipt(indices, seeds) == T
```

Output:

```text
True
```

This confirmed that the recovered indices generated the expected target receipt.

## 5. Capsule Decryption

With the correct indices and lane seeds recovered, the receipt key could be derived.

The key derivation process uses:

```text
SHA256(
    b'Epitaph/v7/capsule\0'
    + lane-wise indices
    + corresponding labels
)
```

The encrypted capsule contained:

- 12-byte nonce
- 59-byte ciphertext and authentication tag
- 43-byte encrypted plaintext
- 16-byte authentication tag

AES-CTR decryption was performed using the derived key.

The initial counter was:

```text
nonce || 0x00000002
```

The decryption was performed using Python's `cryptography` library.

```bash
python3 -I -c "
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

# Derive the receipt key using recovered indices and seeds.
# Decrypt the capsule using AES-CTR.
"
```

## 6. Flag

<pre>
SecLeaf{m0dular_c4rr13s_d0nt_r3st_1n_p34c3}
</pre>

## 7. Key Takeaways

- Recovered a hidden affine transformation using modular arithmetic.
- Identified poisoned calibration records through consistency filtering.
- Applied inverse modular arithmetic and bit rotations.
- Used Wagner's Generalized Birthday Algorithm to solve an 8-list modular sum problem.
- Recovered the receipt key and decrypted the AES-CTR capsule.

## 8. Challenge Summary

| Field | Result |
|---|---|
| Challenge | Eightfold Epitaph |
| Primary Technique | Wagner's Generalized Birthday Attack |
| Mathematical Concept | Modular Arithmetic |
| Additional Concepts | SHA-256, Affine Transformation, Bit Rotation |
| Encryption | AES-CTR |
| Result | Successfully Solved |

**Team HACKBYTES**
