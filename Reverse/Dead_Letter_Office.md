# Dead Letter Office

**Category:** Reverse Engineering  
**Event:** SecLeaf CTF  
**Author:** 5h1kh4r  
**Team:** HACKBYTES

## 1. Reconnaissance

Identified the provided files:

```bash
file *
```

```text
postcard.grid: data
dead_letter_office: ELF 64-bit LSB pie executable, x86-64, dynamically linked, stripped
```

Extracted binary strings:

```bash
strings dead_letter_office
```

Interesting strings:

```text
Delivered.
Returned to sender.
postcard.grid
DLO2
```

The binary reads a 40-character input and validates it using `postcard.grid`.

## 2. Analyzing postcard.grid

Identified the custom file structure:

| Offset | Description |
|---|---|
| `0x00` | Magic: HPL2 |
| `0x04` | Input length: 40 |
| `0x06` | Number of cells: 20 |
| `0x08` | FNV-1a checksum |
| `0x0C` | Initial seed: `0x39c6` |
| `0x80` | Cell data (20 × 4 bytes) |

Verified the FNV-1a checksum:

```text
Expected: 0x41cbd9ec
Result:   0x41cbd9ec
Status:   Valid
```

## 3. Reverse Engineering

Static analysis using `objdump` revealed a custom 14-round Feistel network.

Important parameters:

```text
Input length : 40 bytes
Cells        : 20
Rounds       : 14
Key length   : 8 bytes
Initial seed : 0x39c6
Step         : 7
```

The binary processes input in pairs using XOR, addition and 8-bit circular rotations.

The expected output can be calculated directly from the postcard data:

\[
M=(ROL_{16}(s,3)\oplus((i+1)\times0x9e37))\oplus cell
\]

Since the expected Feistel output was already known, the transformation could be reversed.

Original Feistel equation:

\[
R_{k+1}=F_k(R_k)\oplus R_{k-1}
\]

Reversed equation:

\[
R_{k-1}=F_k(R_k)\oplus R_{k+1}
\]

This allowed recovery of the original input without brute-forcing the characters.

## 4. Solving

Created a Python solver:

```bash
python3 solve.py
```

The script:
- Parses the postcard file.
- Verifies the FNV-1a checksum.
- Extracts the seed, cells and key.
- Reverses all 14 Feistel rounds.
- Reconstructs the original 40-byte input.
- Verifies the recovered values using forward assertions.

Output:

```text
40 20 0x41cbd9ec 0x39c6 208 208
fnv 0x41cbd9ec ok

bytearray(b'SecLeaf{th3_p0st0ff1c3_h4s_4_d4rk_s1d3!}')
```

## 5. Flag Verification

Executed the recovered input against the original binary:

```bash
./dead_letter_office "SecLeaf{th3_p0st0ff1c3_h4s_4_d4rk_s1d3!}"
```

Output:

```text
Delivered.
```

## 6. Final Flag

```text
SecLeaf{th3_p0st0ff1c3_h4s_4_d4rk_s1d3!}
```

## 7. Key Techniques

- ELF Static Analysis
- Custom Binary Format Analysis
- FNV-1a Checksum
- Feistel Network Reversal
- XOR and Bitwise Rotations
- Python Automation

**Team: HACKBYTES**
