# Null Choir

**Category:** Reverse Engineering  
**Event:** SecLeaf CTF  
**Team:** HACKBYTES

## 1. Reconnaissance

Identified the provided files:

```bash
ls
file *
```

```text
canticle.score: data
echoes.trace:   data
null_choir:     ELF 64-bit LSB pie executable, x86-64, dynamically linked, stripped
```

Tested the binary:

```bash
./null_choir
```

Input:
```text
SecLeaf{test}
```

Output:
```text
Dissonance.
```

The binary reads two external files: `canticle.score` and `echoes.trace`. A valid 48-byte input is required to reach `Harmony.`.

## 2. Static Analysis

Generated disassembly:

```bash
objdump -d -M intel --no-show-raw-insn null_choir > dis.txt
```

Identified the following file structures:

**canticle.score**
- Magic: `NCS2`
- Number of instructions: `0x1000` (4096)
- Header size: `0x100`
- Record size: `0x30` (48 bytes)
- Total file size: `196864` bytes

**echoes.trace**
- Magic: `ECH8`
- Record size: 16 bytes
- 32 valid distinct indices required.
- FNV-1a checksum verification.
- Duplicate indices are skipped.

## 3. Recovering the VM Seed

The binary derives a fixed seed from the two files using a chain hash.

The hash function was identified as the **SplitMix64 finalizer**, using these constants:

```text
0xbf58476d1ce4e5b9
0x94d049bb133111eb
```

Other important constants:

```text
G = 0x9e3779b97f4a7c15
K = 0xd1b54a32d192ed03
```

The recovered seed and bank were:

```text
S = 0x866a952e86bb7d8b
m = 0
```

Since the seed depends only on the provided files and not on user input, the complete VM execution could be reconstructed offline.

## 4. Reverse Engineering the Virtual Machine

The binary implements a custom virtual machine with:

- 6 × 64-bit registers.
- 4096 encrypted instruction records.
- A 4-round Feistel permutation.
- Key-dependent instruction decryption.
- 24 custom opcodes.
- A diffusion operation after every 64 instructions.

The instruction records were decrypted using the recovered seed.

Each instruction contained an opcode, register operands, rotation value and immediate value.

The opcode was calculated as:

```python
opcode = (op_raw - 7*i - S_byte[i & 7]) & 0x1f
```

Values greater than or equal to 24 were treated as NOP instructions.

**Important discovery:** Two jump-table entries contained overlapping instructions. Normal disassembly initially misrepresented these operations.

The actual operations were:

```text
Opcode 6: R[A] ^= imm
Opcode 7: R[A] += imm
```

## 5. Validating the VM Model

Created a forward VM implementation in `vm.py`.

```bash
python3 vm.py
```

Output:

```text
S 0x866a952e86bb7d8b m 0 bad 0
```

All 4096 instruction records passed their structural checks.

The implementation was also compared against the original binary using GDB.

Both produced identical register states byte-for-byte, confirming the reconstructed VM logic.

## 6. Reversing the VM

The final target state was extracted from `canticle.score`.

Instead of brute-forcing the input, the VM was executed backwards.

The inversion process:

1. Initialize registers with the target state.
2. Iterate through instructions from 4095 down to 0.
3. Reverse diffusion operations after every 64 instructions.
4. Apply the inverse of each opcode.
5. Reverse the initial register key schedule.
6. Verify the result using forward and backward execution.

The recovered input was:

```bash
python3 inv.py
```

Output:

```text
b'SecLeaf{th3_ch01r_s1ng5_1n_5h4d0w_m4ch1n35!!!!!}'
```

## 7. Flag Verification

Submitted the recovered input to the original executable:

```bash
./null_choir < ans.txt
```

Output:

```text
Harmony.
```

<badge color="success">Successfully solved</badge>

## 8. Final Flag

```text
SecLeaf{th3_ch01r_s1ng5_1n_5h4d0w_m4ch1n35!!!!!}
```

## 9. Key Techniques

- ELF Static Analysis
- Custom Virtual Machine Reverse Engineering
- SplitMix64
- Feistel Network
- Instruction Decryption
- Opcode Reversal
- Register State Analysis
- Python Automation

**Team: HACKBYTES**
