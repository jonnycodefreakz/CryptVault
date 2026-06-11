# 🔐 CryptVault 2.0

### AES-256-GCM · Argon2id · Hardware-Bound · Anti-Forensic · Air-Gapped

**Just your files, encrypted, on your machine. No network calls. No telemetry. No leakage.**

---

## 📂 What It Does

CryptVault creates a local encrypted vault on Windows. Drag files in and they're encrypted. Select files and decrypt them back out. The vault locks itself when you step away.

Nothing touches the internet — the application is fully air-gapped, verified by independent sandbox analysis. No accounts. No servers. No subscriptions.

---

## 🛡️ What Makes It Secure

### 🔒 Encryption
Every file is encrypted with **AES-256-GCM**. This provides both secrecy and tamper detection — if an encrypted file is modified, decryption is refused.

### 🔑 Password Protection
Your password is processed through **Argon2id with 128MB of memory**, making brute-force attacks painfully expensive even with specialized hardware.

### 🧂 Pepper
A unique **256-bit pepper secret** is generated once per machine and stored outside the vault directory. The vault alone is useless without it.

### 🖥️ Hardware Binding
The vault is **bound to your specific hardware** — CPU, motherboard, BIOS. Even with the password, files won't decrypt on a different PC.

### 📇 Metadata Protection
Salts, tokens, recovery data, and the filename index are **encrypted with a device-bound key**. File names on disk are random hex strings.

### 🧠 Memory Protection
The encryption key in memory is **DPAPI-protected** and cryptographically zeroed the moment the vault locks.

### 🚫 Brute-Force Defence
After **5 wrong password attempts**, an exponential lockout activates — escalating from 30 seconds to a full hour.

### 💀 Deletion
Every deletion is a **7-pass random overwrite** before the file is removed — not a simple delete.

### ☠️ Emergency Nuke
A **3-second hold** on the nuke button destroys the entire vault. All files shredded. All metadata destroyed. No dialogs. Release to cancel.

---

## ✅ Independent Security Verification

| Scanner | Result | Link |
|---------|--------|------|
| 🛡️ **VirusTotal** | 0/72 detections | [View Report](https://www.virustotal.com/gui/file/f2d1edb16f66cd12e4643fe4af661dc6a494995c2f97dcdf2af55884a3cd56c5/detection) |
| 🔬 **Hybrid Analysis** | No malicious indicators | [View Report](https://hybrid-analysis.com/file-collection/6a2ad84aa9ade25f360c8b6c) |

**Both scans confirm zero network activity and no malicious behaviour.**

---

## ⚠️ What It Cannot Protect Against

CryptVault runs in user mode on Windows. It trusts the operating system.

- 🦠 **Compromised OS** — kernel-level malware or keyloggers are outside any application's reach
- 🧊 **DMA attacks & cold boot extraction** — requires physical access and specialised hardware
- 🔌 **Hardware implants** — outside the threat model of any software-only tool
- 🔓 **Weak passwords** — encryption is only as strong as the password protecting it
- 💾 **SSD wear-leveling** — shredding cannot fully guarantee destruction on solid-state drives (use BitLocker)
- 📋 **Volume Shadow Copies** — Windows snapshots may retain file copies
- ☁️ **Cloud sync folders** — OneDrive, Dropbox, and similar may retain online copies
- 💤 **Hibernation & swap files** — may contain fragments of decrypted data

**This is one layer in a defence-in-depth strategy. Pair it with full-disk encryption, keep your system clean, and use a strong unique password.**

---

## 📁 Code Screenshots

The application is Open Core source code

- Key derivation (Argon2id + pepper)
- File encryption (AES-256-GCM)
- Machine binding (SHA-512 hardware fingerprint)
- Brute-force protection (exponential lockout)
- Anti-forensic shredding (7-pass overwrite)
- Memory protection (cryptographic zeroing)
- Metadata encryption (device-bound key)
- Emergency nuke (3-second hold)

---

## 📸 Screenshots

![Vault locked](https://github.com/user-attachments/assets/0feee348-d796-48c8-b1b1-2cafdea3bf11)

![Vault unlocked with files](https://github.com/user-attachments/assets/6a71a4d6-b40e-46fb-bb81-4d679a0c37eb)

![Security info dialog](https://github.com/user-attachments/assets/9f2ac240-9eb3-48d4-b9d9-b1dcfaf63faf)

---

## 💬 Feedback

Cryptographic code needs outside eyes. If you find a bug, a vulnerability, a design flaw, or anything that doesn't feel right — **open an issue**.

All feedback matters. Especially the critical kind. Thanks everyone
