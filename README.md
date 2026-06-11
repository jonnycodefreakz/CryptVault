# CryptVault
CryptVault 2.0
AES-256-GCM · Argon2id · hardware-bound · anti-forensic · zero cloud

Just your files, encrypted, on your machine.

What It Does
CryptVault creates an encrypted vault on your Windows PC. Drag files in and they're encrypted. Select files and they decrypt back out. The vault locks when you walk away. Nothing goes to the cloud. There are no accounts, no servers, and no subscriptions.

What Makes It Strong
Files are encrypted with AES-256-GCM. This means the data is scrambled and any tampering gets detected. Your password runs through Argon2id with 128 megabytes of memory, making brute-force attacks painfully expensive even with specialized hardware. A secret pepper unique to your machine means the vault directory is useless on its own. The vault is bound to your hardware. Even with your password, it won't decrypt on a different PC.

Metadata, salts, tokens, and the file name index are encrypted separately with a device-bound key. File names on disk are random hex strings. The encryption key in memory is DPAPI-protected and zeroed when the vault locks. After five wrong password attempts, an exponential lockout kicks in.

Every file deletion uses a 7-pass random overwrite before the file is removed.

An emergency nuke button destroys everything on a 3-second hold. No dialogs. Release to cancel.

What It Can't Do
CryptVault runs in user mode on Windows. It trusts the operating system. If Windows is compromised by kernel-level malware or a keylogger, there's nothing any application can do about it. DMA attacks, cold boot memory extraction, and hardware implants are outside its reach.

A weak password can still be guessed. Shredding has limits on SSDs due to wear-leveling. Volume Shadow Copies, cloud sync folders, hibernation files, and swap files can retain plaintext outside the vault.

This is one layer of protection. Pair it with BitLocker, keep your system clean, and use a strong unique password.

Feedback
This is a personal project, but cryptographic code needs outside eyes. If you find a bug, a security flaw, a design problem, or something that doesn't feel right, open an issue. All feedback matters, especially the critical kind.

Completly safe and tested

VirusTotal.com
https://www.virustotal.com/gui/file/f2d1edb16f66cd12e4643fe4af661dc6a494995c2f97dcdf2af55884a3cd56c5/detection

Hybrid Analysis
http://hybrid-analysis.com/file-collection/6a2ad84aa9ade25f360c8b6c






<img width="1182" height="866" alt="Screenshot 2026-06-11 160545" src="https://github.com/user-attachments/assets/0feee348-d796-48c8-b1b1-2cafdea3bf11" />

<img width="1182" height="876" alt="Screenshot 2026-06-11 161039" src="https://github.com/user-attachments/assets/6a71a4d6-b40e-46fb-bb81-4d679a0c37eb" />

<img width="1181" height="871" alt="Screenshot 2026-06-11 160957" src="https://github.com/user-attachments/assets/9f2ac240-9eb3-48d4-b9d9-b1dcfaf63faf" />
