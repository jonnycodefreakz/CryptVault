# 🔐 Security Architecture

This project is personal, but the cryptographic design aims for practical defense-in-depth.

---

## File Encryption — AES-256-GCM

Each encrypted file receives a unique **96-bit nonce** and **128-bit authentication tag**.

* AES-256-GCM provides **confidentiality + integrity**
* Authentication tags detect modification or corruption
* Nonce reuse is prevented by generating secure random values per file
* Plaintext buffers are wiped after encryption

```csharp
private void EncryptFile(string src)
{
    byte[] key = GetActiveKey();
    byte[] nonce = RandomNumberGenerator.GetBytes(12);

    byte[] pt = File.ReadAllBytes(src);
    byte[] ct = new byte[pt.Length];
    byte[] tag = new byte[16];

    using AesGcm g = new AesGcm(key, 16);
    g.Encrypt(nonce, pt, ct, tag);

    using FileStream fs = File.Create(dest);
    fs.Write(nonce, 0, 12);
    fs.Write(tag, 0, 16);
    fs.Write(ct, 0, ct.Length);

    CryptographicOperations.ZeroMemory(pt);
}
```

---

## Device Binding — Hardware-Derived Key Protection

Encryption keys are additionally bound to a hardware fingerprint.

Even with the correct password, encrypted content cannot be directly moved and decrypted elsewhere without reproducing the expected environment.

```csharp
private byte[] GetMachineBoundKey(byte[] baseKey)
{
    byte[] h = SHA512.HashData(
        Encoding.UTF8.GetBytes(GetMachineIdentifier()));

    byte[] b = new byte[baseKey.Length];

    for (int i = 0; i < baseKey.Length; i++)
        b[i] = (byte)(baseKey[i] ^ h[i % h.Length]);

    return b;
}

private string GetMachineIdentifier() =>
    $"{GetCpuId()}|{GetBoard()}|{GetBios()}|{Environment.MachineName}|{GetOs()}";
```

---

## Brute-Force Resistance — Exponential Lockout

Failed unlock attempts trigger escalating delays.

Features:

* Lockout begins after **5 failed attempts**
* Delay doubles progressively
* Maximum lockout: **1 hour**
* State persists securely across restarts using **DPAPI**

```csharp
private void RecordFail()
{
    _failedAttempts++;

    if (_failedAttempts >= 5)
    {
        _lockoutDurationSeconds =
            Math.Min(
                30 * (int)Math.Pow(2, _failedAttempts - 5),
                3600);

        _lockoutUntil =
            DateTime.Now.AddSeconds(_lockoutDurationSeconds);
    }

    SaveBF();
}
```

---

## Installation Pepper — DPAPI-Protected Secret

A unique **256-bit installation pepper** is generated once per machine.

This adds another layer of protection beyond passwords and vault files.

```csharp
private static byte[] GenerateInstallationPepper()
{
    string path = Path.Combine(
        Environment.GetFolderPath(
            Environment.SpecialFolder.LocalApplicationData),
        "Microsoft",
        "Credentials",
        ".cv_pepper");

    if (File.Exists(path))
    {
        byte[] existing =
            ProtectedData.Unprotect(
                File.ReadAllBytes(path),
                null,
                DataProtectionScope.CurrentUser);

        if (existing?.Length == 32)
            return existing;
    }

    byte[] pepper =
        RandomNumberGenerator.GetBytes(32);

    File.WriteAllBytes(
        path,
        ProtectedData.Protect(
            pepper,
            null,
            DataProtectionScope.CurrentUser));

    File.SetAttributes(
        path,
        FileAttributes.Hidden |
        FileAttributes.System);

    return pepper;
}
```

---

## Secure Deletion — Multi-Pass Overwrite

Files are overwritten with cryptographic randomness before deletion.

```csharp
private void ShredFile(string path)
{
    long len = new FileInfo(path).Length;

    using FileStream fs =
        new FileStream(
            path,
            FileMode.Open,
            FileAccess.Write);

    byte[] rnd = new byte[4096];

    for (int p = 0; p < 7; p++)
    {
        fs.Seek(0, SeekOrigin.Begin);

        long rem = len;

        while (rem > 0)
        {
            RandomNumberGenerator.Fill(rnd);

            int w =
                (int)Math.Min(
                    rem,
                    rnd.Length);

            fs.Write(rnd, 0, w);

            rem -= w;
        }

        fs.Flush();
    }

    fs.SetLength(0);
    fs.Close();

    File.Delete(path);
}
```

---

## Memory Protection — Secure Key Zeroization

Vault keys are actively cleared from memory when the vault locks.

```csharp
private void ClearVaultKey()
{
    if (_key != null)
    {
        CryptographicOperations.ZeroMemory(_key);
        _key = null;
    }

    if (_encryptedKey != null)
    {
        CryptographicOperations.ZeroMemory(_encryptedKey);
        _encryptedKey = null;
    }

    if (_keyEntropy != null)
    {
        CryptographicOperations.ZeroMemory(_keyEntropy);
        _keyEntropy = null;
    }
}
```

---

## Metadata Protection — Device-Bound Encryption

Metadata such as:

* salts
* tokens
* recovery data
* filename index

is encrypted separately using a device-derived key.

```csharp
private byte[] ProtectMetadata(byte[] plain)
{
    byte[] dk = GetDeviceBoundKey();

    byte[] n =
        RandomNumberGenerator.GetBytes(12);

    byte[] ct =
        new byte[plain.Length];

    byte[] tag =
        new byte[16];

    using AesGcm g =
        new AesGcm(dk, 16);

    g.Encrypt(
        n,
        plain,
        ct,
        tag);

    byte[] r =
        new byte[
            12 + 16 + ct.Length];

    Array.Copy(n, 0, r, 0, 12);
    Array.Copy(tag, 0, r, 12, 16);
    Array.Copy(ct, 0, r, 28, ct.Length);

    return r;
}
```

---

## Emergency Nuke — Hold 3 Seconds

Designed for rapid destruction of vault contents.

Actions:

* decrypt metadata
* reveal hidden files
* shred encrypted content
* erase metadata
* clear indexes
* reset lockout state
* wipe active keys

```csharp
private void ExecuteNuke()
{
    UnprotectMetadataFiles();
    UnhideVaultFiles();

    LogActivity(
        "NUKE EXECUTED destroying vault");

    int destroyed = 0;

    foreach (string f in
        Directory.GetFiles(
            _vaultPath,
            "*.enc"))
    {
        try
        {
            ShredFile(f);
            destroyed++;
        }
        catch
        {
            try
            {
                File.Delete(f);
                destroyed++;
            }
            catch { }
        }
    }

    foreach (string mf in
        Directory.GetFiles(_metaPath))
    {
        try
        {
            ShredFile(mf);
        }
        catch
        {
            try
            {
                File.Delete(mf);
            }
            catch { }
        }
    }

    _fileIndex.Clear();

    SaveFileIndex();

    ResetBF();

    ClearVaultKey();
}
```

---

## Feedback

Cryptographic code benefits from external review.

If you find:

* security issues
* implementation bugs
* architectural concerns
* edge cases
* cryptographic mistakes

please open an issue.

Critical feedback is especially valuable.
