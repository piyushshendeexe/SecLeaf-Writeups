┌──(hackbytes㉿hackbytes)-[~]
└─$ ls
canticle.score  echoes.trace  null_choir

┌──(hackbytes㉿hackbytes)-[~]
└─$ file *
canticle.score: data
echoes.trace:   data
null_choir:     ELF 64-bit LSB pie executable, x86-64, dynamically linked, stripped

┌──(hackbytes㉿hackbytes)-[~]
└─$ ./null_choir
SecLeaf{test}
Dissonance.

# stripped PIE, symbols nahi. strings: "Harmony." / "Dissonance." / "rb"
# "canticle.score" / "echoes.trace" / "silence.\n" (stderr, file na mile to)
# yaani binary khud 2 files padhti hai, aur sirf win/lose print karti hai

┌──(hackbytes㉿hackbytes)-[~]
└─$ objdump -d -M intel --no-show-raw-insn null_choir > dis.txt

# ---------- main() ka structure ----------
# 1. input 48 bytes (strlen == 0x30) -> 6 x u64 registers R[0..5]
# 2. canticle.score:  magic "NCS2", N = 0x1000 (u32 @ +4), hdr u64 @ +8
#                     size == 0x100 + 0x30*N  -> 196864 ✓
# 3. echoes.trace:    magic "ECH8", 16-byte records "EC" | tag | idx | val(8) | fnv32
#    - tag == byte5 of score hdr (0xe5), idx <= 0x1f
#    - FNV-1a(rec[0:12]) == rec[12:16], pehle wala idx jeetta hai (dupes skip)
#    - 32 valid distinct idx chahiye warna fail  -> baaki records DECOYS
#    - E[idx] = F((idx+1)*G ^ hdr ^ 0xe0c4..a366) ^ val
# 4. chain hash: h = F(E[i] ^ rcx ^ h), rcx += K   (32 rounds, 33*K == const check)
# 5. SEED  S = F(h ^ key1 ^ hdr ^ key2 ^ 0xe66cd00b5a17e123)
#    ^ S input pe depend NAHI karta, sirf files pe. matlab VM fixed hai.
#    F = splitmix64 finalizer (0xbf58476d1ce4e5b9, 0x94d049bb133111eb, shifts 30/27/31)
#    G = 0x9e3779b97f4a7c15 , K = 0xd1b54a32d192ed03

# ---------- VM ----------
# bank   m = (S>>11) % 3          -> score me 3 banks, ek hi use hota hai
# order  step i ka record = perm(i)  (4-round Feistel on 6|6 bits, round fn = F(x^rcx^S)&0x3f)
# decrypt rec[j] ^= F(K*(j+1) ^ (S + G*(i+1))) >> ((j&7)*8)
# plaintext:  [0]=op_raw [1]=A [2]=B [4]=rot [5]==i&0xff [6]==0x7a [7]==0xa7 [8:16]=imm
# opcode = (op_raw - 7*i - S_byte[i&7]) & 0x1f      (>=24 -> NOP)
# har 64 step ke baad ek diffusion round:
#     for j in 0..5: x = R[j] ^ F(j*G ^ S ^ i);  R[(j+2)%6] += F(x)
# input load:  R[j] = rol(R[j] ^ F(S ^ G*(j+1)), 14+7j)
# end: R bytes ^ keystream(S) == score[0x20:0x50]  -> "Harmony."

# jump table @0x2060 (24 entries). 2 entries INSTRUCTION ke beech me jump karti hain:
#    entry 6 -> 0x1892  ("xor [R+A*8],rdx" ,  `and rdx,..` skip)   => R[A] ^= imm
#    entry 7 -> 0x186a  ("add [R+A*8],rdx" ,  rol skip)             => R[A] += imm
# disassembler me overlapping bytes dikhte hain, yahi trap hai

 op  semantics                        op  semantics
  0  A ^= imm+B                       12  A = ~(A^imm)
  1  A += rol(imm^B,r)                13  A = bitrev(A)
  2  A ^= rol(imm+B,r)                14  A = rol(imm^A,r)
  3  (A,B) = (B, A^F(imm+B))          15  A = (imm^A)+rol(B,r)
  4  swap(A,B)                        16  A ^= F(imm^B)
  5  A = rol(A,r)                     17  A += F(imm+B)
  6  A ^= imm                         18  A = rol(imm+A,r)
  7  A += imm                         19  A = rol(A,r) ^ F(imm+B)
  8  A = bswap(A)                     20  A ^= imm & B
  9  A = A^B^imm                      21  A += rol(imm,r)^B
 10  A -= rol(imm^B,r)                22  A = ~rol(A^F(imm),r)
 11  lanes rotate by r%6              23  A ^= rol(F(imm^B),r)

# ---------- idea ----------
# S input se independent => poora program fixed => har op INVERTIBLE hai
# (A==B sirf 1-register ops me aata hai: 5,6,7,8,11,12,13,14,18,22)
# plan: file se S nikalo, 4096 records decrypt karo, target state se VM ULTA chalao

┌──(hackbytes㉿hackbytes)-[~]
└─$ python3 vm.py            # forward model
S 0x866a952e86bb7d8b m 0 bad 0
target 8e6cebd3b8cdeee9632a897627f5932ab6cb8ef28095ec41602c67eb43b4db7edd8f7691383f3c46c93af977b67a3a24
...
# bad 0 -> 4096 ke 4096 records ke 7a/a7/idx checks pass: S, perm, keystream sahi

# model ko asli binary se verify karo: gdb me end-of-loop (base+0x1cc6) pe R dump
┌──(hackbytes㉿hackbytes)-[~]
└─$ gdb -q -batch -x g.cmd ./null_choir < in.txt      # input = 'A'*48
0x7fffffff9240: 0xfd 0x8a 0x37 0xca 0x3f 0x2b 0x0c 0x22 ...
┌──(hackbytes㉿hackbytes)-[~]
└─$ python3 vm.py AAAA...    # mera model
fd8a37ca3f2b0c22d017a51fd0c55b1cdbab0d825c2efbf0a534c8ffbcc14f45e9393867...
# byte-for-byte match ✓

# ---------- solve ----------
# inv.py:  R = target
#   for i = N-1 .. 0:  (i&63==63 -> undo diffusion, reverse j=5..0)  ; undo op[i]
#   undo key schedule: R[j] = ror(R[j],14+7j) ^ F(S ^ G*(j+1))
# + random input se forward->backward round-trip assert

┌──(hackbytes㉿hackbytes)-[~]
└─$ python3 inv.py
b'SecLeaf{th3_ch01r_s1ng5_1n_5h4d0w_m4ch1n35!!!!!}'

┌──(hackbytes㉿hackbytes)-[~]
└─$ ./null_choir < ans.txt
Harmony.

Flag: SecLeaf{th3_ch01r_s1ng5_1n_5h4d0w_m4ch1n35!!!!!}
