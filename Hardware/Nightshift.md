
┌──(hackbytes㉿hackbytes)-[~]
└─$ unzip Eightfold_Epitaph.zip && file *
data.json: JSON data
model.py:  Python script

# scenario: "Eightfold Funeral: Epitaph". 8 lanes, har lane me 2^21 virtual labels, sum mod 2^84 (modular addition) se target banana hai.
# capsule (nonce + ct_tag) me flag band hai, key = receipt_key(indices, seeds)
# 2 files: model.py (public arithmetic) + data.json (calibration, lane_stamps, target_stamp, capsule)
# python3 + numpy + cryptography se offline solve hua.

# ---------- recon (model.py) ----------
# MOD = 2^84, SIZE = 2^21
# stamp(x) = rol84(m*x + c, r)        -> m odd, c, r secret (ek hi affine map sabke liye)
# label(seed, lane, i) = sha256(b'Epitaph/v7|' + lane + seed(11B LE) + i(4B LE))[:11] % 2^84
# receipt = sum of 8 labels mod 2^84  (lane 0..7 me se ek ek)
# receipt_key = sha256(b'Epitaph/v7/capsule\0' + for each lane: index(4B) + label(11B))
# data.json: 25 calibration records, 8 lane_stamps, 1 target_stamp, capsule (12B nonce, 59B ct_tag)
# hint: "Calibration records are not guaranteed honest" -> poisoned/noisy data present hai.

# ---------- step 1: secret affine map recover karna ----------
# har r in 0..83 ke liye stamp ko ror kiya -> y = m*x + c mod 2^84
# diff dy = m * dx, dy * inv(dx) mod 2^84 se m recover kiya aur c find kiya:
#   r=35 : m=0x750e59bef1f0e6b385693, c=0x67cd6060c7616d28d7c34 -> 15 inliers (honest map)
#   r=71 : m=0x78e4a9198f5ceca64eb75, c=0x3b4224d942b574c008800 -> 7 inliers (decoy map)
#   baaki 3 records pure noise hain.

# ---------- step 2: stamps invert ----------
# unstamp(s) = (ror(s, 35) - c) * inv(m) mod 2^84
# 8 lane seeds + target T = 0x1b59868b301e63f23871f calculate ho gaya.

# ---------- step 3: generalized birthday problem (Wagner k=8 over Z/2^84) ----------
# 8 lists x 2^21 labels, condition: sum == T mod 2^84.
# n / (1 + log2(k)) = 84 / 4 = 21 bits per level (exact match for 2^21 size):
#   Level 1: (0,1), (2,3), (4,5), (6,7) merge kiye on low 21 bits (with carry handling)
#   Level 2: 4 lists ko merge kiya on low 42 bits
#   Level 3: Final 2 lists match karke canonical solution find kiya:
# indices = [1924587, 2031184, 29597, 785188, 676753, 560734, 264902, 1161509]
# model.receipt(indices, seeds) == T -> True

# ---------- step 4: capsule decrypt ----------
# key = receipt_key(indices, seeds)
# nonce = 12B, ct_tag = 59B (43B ct + 16B tag)
# AES-CTR keystream use karke decrypt kiya (initial counter = nonce || 0x00000002):

┌──(hackbytes㉿hackbytes)-[~]
└─$ python3 -I -c "
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
# key derivation via receipt_key
# ciphertext decryption via AES-CTR (counter start = 2)
"
b'SecLeaf{m0dular_c4rr13s_d0nt_r3st_1n_p34c3}'

# ---------- IOCs / summary ----------
# map params    : r=35, m=0x750e59bef1f0e6b385693, c=0x67cd6060c7616d28d7c34
# target        : T = 0x1b59868b301e63f23871f
# vulnerability : Wagner's Generalized Birthday Attack on modular addition with calibration filtering

Flag: SecLeaf{m0dular_c4rr13s_d0nt_r3st_1n_p34c3}
