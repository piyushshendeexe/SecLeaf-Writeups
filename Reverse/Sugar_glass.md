# Sugar Glass

**Category:** Reverse Engineering  
**Event:** SecLeaf CTF  
**Team:** HACKBYTES

## Challenge Overview

The challenge provides a stripped 64-bit ELF executable named `sugar_glass`.

The objective was to reverse engineer its validation logic, understand the custom byte transformations, recover the expected 29-character input, and successfully validate the recovered flag.

## 1. Binary Reconnaissance

First, identified the executable format:

```bash
file sugar_glass
```

Output:

```text
ELF 64-bit LSB pie executable, x86-64,
dynamically linked, stripped
```

Made the binary executable:

```bash
chmod +x sugar_glass
```

Next, examined the available strings:

```bash
strings -n4 sugar_glass
```

Interesting output:

```text
fgets
strlen
strcspn
strncpy
Filed.
Rejected.
SGR2
```

The strings indicated an input-validation mechanism with two possible outcomes.

## 2. Static Analysis

Used `objdump` to inspect the disassembly:

```bash
objdump -d -M intel sugar_glass | less
```

The important functions identified were:

- `main()` at `0x1090`: Reads and processes user input.
- `check()` at `0x1230`: Performs the actual validation.

The validation function returns:

- `1` → `Filed.`
- `0` → `Rejected.`

The binary's `.rodata` section was examined to extract the embedded constants:

```bash
objdump -s -j .rodata sugar_glass
```

The `SGR2` marker was identified along with the encrypted byte array and validation table.

## 3. Understanding the Validation Algorithm

Further disassembly analysis revealed that the program validates a 29-character input using a custom byte-processing algorithm.

The important parameters were:

| Parameter | Value |
|---|---|
| Required input length | 29 bytes |
| Rotation increment | 11 |
| Initial seed | 0x05 |
| Initial accumulator | 0x5D |
| Initial byte state | 0x42 |
| Initial secondary state | 0x39 |
| Validation table size | 29 bytes |

The validation process involves:

1. Selecting a scrambled input position using `esi % 29`.
2. XORing the selected character with the corresponding ciphertext byte.
3. Adding the current state value.
4. Applying an 8-bit circular left rotation by 3.
5. XORing with a secondary state.
6. Applying another 8-bit circular left rotation by 1.
7. Updating the accumulator and comparing its lower byte against the validation table.

The input position changes by 11 during every iteration, creating a scrambled mapping between input characters and validation rounds.

## 4. Developing the Solver

Instead of manually reversing all 29 characters, developed a Python script to reproduce the binary's validation logic.

The script implements:

- An 8-bit rotate-left function.
- Ciphertext and validation-table extraction.
- Scrambled index calculation.
- State and accumulator updates.
- Printable ASCII character testing.

The core rotation function:

```python
def rol8(x, n):
    x &= 0xff
    return ((x << n) | (x >> (8 - n))) & 0xff
```

The solver iterates through printable ASCII characters and checks each candidate against the corresponding validation condition.

The recovered characters are placed into their original positions using the scrambled index calculation.

The complete implementation was saved as:

```text
solve.py
```

## 5. Recovering the Flag

Executed the solver:

```bash
python3 solve.py
```

Recovered output:

```text
SecLeaf{gl455_h0ld5_4_53cr3t}
```

## 6. Flag Verification

Finally, tested the recovered flag against the original executable:

```bash
echo "SecLeaf{gl455_h0ld5_4_53cr3t}" | ./sugar_glass
```

Output:

```text
Filed.
```

The binary accepted the recovered input, confirming the successful solution.

## 7. Final Flag

```text
SecLeaf{gl455_h0ld5_4_53cr3t}
```

## 8. Key Takeaways

- ELF binary reconnaissance using `file` and `strings`.
- Static analysis using `objdump`.
- Understanding stripped ELF executables.
- Reverse engineering custom validation routines.
- Bitwise operations, XOR, and circular rotations.
- Reproducing binary logic using Python.
- Recovering and validating an obfuscated input.

**Team: HACKBYTES**
