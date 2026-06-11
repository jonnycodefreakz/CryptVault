File Encryption — AES-256-GCM
Every file gets a unique 96-bit nonce. The authentication tag detects any tampering:
---------------------------------------------------------------------------------
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


Machine Binding — Hardware Fingerprint
The derived key is XORed with a SHA-512 hash of your specific hardware. Even with the correct password, files won't decrypt on a different machine:
----------------------------------------------------------------------------------------------------------------------------------------------------

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

    
Exponential Brute-Force Lockout
After 5 wrong attempts, delays escalate. The lockout state is DPAPI-encrypted and survives restarts:
-----------------------------------------------------------------------------------------------------

private void RecordFail()
{
    _failedAttempts++;
    if (_failedAttempts >= 5)
    {
        _lockoutDurationSeconds = Math.Min(
            30 * (int)Math.Pow(2, _failedAttempts - 5),
            3600); // 30s → 1 hour max
        _lockoutUntil = DateTime.Now.AddSeconds(_lockoutDurationSeconds);
    }
    SaveBF(); // DPAPI-protected, persists across sessions
}


Installation Pepper — DPAPI-Protected Secret
A 256-bit secret is generated once per machine. The vault directory is cryptographically useless without it:
------------------------------------------------------------------------------------------------------------

private static byte[] GenerateInstallationPepper()
{
    string path = Path.Combine(
        Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
        "Microsoft", "Credentials", ".cv_pepper");

    if (File.Exists(path))
    {
        byte[] existing = ProtectedData.Unprotect(
            File.ReadAllBytes(path), null,
            DataProtectionScope.CurrentUser);
        if (existing?.Length == 32) return existing;
    }

    byte[] pepper = RandomNumberGenerator.GetBytes(32);
    File.WriteAllBytes(path, ProtectedData.Protect(
        pepper, null, DataProtectionScope.CurrentUser));
    File.SetAttributes(path,
        FileAttributes.Hidden | FileAttributes.System);
    return pepper;
}


Anti-Forensic Shredding — 7-Pass Overwrite
Every file deletion overwrites with cryptographic randomness before the file is removed:
----------------------------------------------------------------------------------------

private void ShredFile(string path)
{
    long len = new FileInfo(path).Length;
    using FileStream fs = new FileStream(
        path, FileMode.Open, FileAccess.Write);
    byte[] rnd = new byte[4096];

    for (int p = 0; p < 7; p++)
    {
        fs.Seek(0, SeekOrigin.Begin);
        long rem = len;
        while (rem > 0)
        {
            RandomNumberGenerator.Fill(rnd);
            int w = (int)Math.Min(rem, rnd.Length);
            fs.Write(rnd, 0, w);
            rem -= w;
        }
        fs.Flush();
    }
    fs.SetLength(0);
    fs.Close();
    File.Delete(path);
}

Memory Protection Key Zeroed on Lock
When the vault locks, the key is cryptographically zeroed, not left for the garbage collector:
------------------------------------------------------------------------------------------------

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


Metadata Protection — Device-Bound Encryption
Salts, tokens, recovery data, and the filename index are encrypted with a key derived from the machine fingerprint:
-------------------------------------------------------------------------------------------------------------------

private byte[] ProtectMetadata(byte[] plain)
{
    byte[] dk = GetDeviceBoundKey();
    byte[] n = RandomNumberGenerator.GetBytes(12);
    byte[] ct = new byte[plain.Length];
    byte[] tag = new byte[16];

    using AesGcm g = new AesGcm(dk, 16);
    g.Encrypt(n, plain, ct, tag);

    byte[] r = new byte[12 + 16 + ct.Length];
    Array.Copy(n, 0, r, 0, 12);
    Array.Copy(tag, 0, r, 12, 16);
    Array.Copy(ct, 0, r, 28, ct.Length);
    return r;
}


Emergency Nuke — 3-Second Hold to Destroy Everything
No dialogs. No typing. Hold for 3 seconds and the entire vault is shredded:
---------------------------------------------------------------------------

private void ExecuteNuke()
{
    UnprotectMetadataFiles();
    UnhideVaultFiles();

    LogActivity("NUKE EXECUTED destroying vault");

    int destroyed = 0;
    foreach (string f in Directory.GetFiles(_vaultPath, "*.enc"))
    {
        try { ShredFile(f); destroyed++; }
        catch { try { File.Delete(f); destroyed++; } catch { } }
    }

    foreach (string mf in Directory.GetFiles(_metaPath))
    {
        try { ShredFile(mf); } catch { try { File.Delete(mf); } catch { } }
    }

    _fileIndex.Clear();
    SaveFileIndex();
    ResetBF();
    ClearVaultKey();
}

---------------------------------------------------------------------------------------------------------
Feedback
This is a personal project, but cryptographic code needs outside eyes. 
If you find a bug, a security flaw, a design problem, or something that doesn't feel right, open an issue. 
All feedback matters, especially the critical kind.
----------------------------------------------------------------------------------------------------------
