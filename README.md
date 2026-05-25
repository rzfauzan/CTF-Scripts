# CTF Scripts

A collection of automation scripts and exploit templates for **Capture the Flag (CTF)** challenges.
This repository focuses on accelerating workflows during competitions, particularly in the **Binary Exploitation, Cryptography,** and **Digital Forensics categories.**

---

## Tech Stack

- **OS:** Linux (Kali Linux / Debian Recommended)
- **Language:** Python 3
- **Libraries:**
  - `pwntools` — binary exploitation automation

---

## Module Overview

| Module | Description | Technical Focus |
|--------|-------------|-----------------|
| **Binex** | Exploit development for vulnerable binaries | ASLR/NX/PIE Bypass, Canary Leak, ROP, Format String |
| **Crypto** | Cryptanalysis and decryption scripts | AES Modes and RSA Attacks |
| **Forensics** | Digital artifact and traffic analysis | PCAP Analysis, Metadata, Blockchain Trace |

---

## Usage

Most scripts are modular and can be executed directly.

### Example — RSA Hastad Attack
```bash
cd "crypto/rsa attack/hastad attack"
python3 final.py
```

### Example — Remote Binary Exploitation
```bash
python3 exploit.py REMOTE
```

---

## Notes

Each subfolder contains supporting notes such as `.html` or `catatan.txt` files that explain:

- attack theory
- exploitation flow
- step-by-step solving references

This makes the repository useful not only during CTFs, but also for future practice and review.

---

## Disclaimer

This repository is intended for **educational purposes, legal labs, and CTF competitions only**.  
Any misuse outside authorized environments is the sole responsibility of the user.
