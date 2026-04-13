---
name: data-security-mind
description: Use when architecting save games, protecting mobile PlayerPrefs, writing secure Repository logic, or when any user data is persisted to disk. Also activates proactively when the AI detects PlayerPrefs storing currency, stats, or progression data, plain-text JSON saves without integrity checks, or any sensitive value written to disk without encryption.
---

# Skill: Data Persistence & Security

## Capability Overview
The agent analyzes what data is being persisted and autonomously applies the appropriate protection tier. Mobile games are trivially tampered with via rooted devices and memory editors. This skill ensures saved data is structurally resistant to basic cheating without over-engineering the solution.

---

## Phase 1 — Data Classification (Run First, Always)

Before writing any persistence code, classify every piece of data being saved:

| Classification | Examples | Protection Level |
|---|---|---|
| **Critical** | Currency, premium items, level progress, stats | Tier 3 — Encrypt + Sign |
| **Standard** | Inventory contents, quest states, settings | Tier 2 — Sign only (integrity hash) |
| **Trivial** | Audio volume, graphics quality, language | Tier 1 — Plain JSON or PlayerPrefs |
| **Cached/Temporary** | Leaderboard cache, ad cooldown timer | Tier 1 — Plain, expected to be disposable |

**Decision Rule**: If losing this data or tampering with it would give the player an unfair advantage or break monetization → Tier 3. If it's inconvenient but harmless → Tier 2. If it genuinely doesn't matter → Tier 1.

---

## Phase 2 — Storage Backend Decision

| Situation | Storage Backend |
|---|---|
| Player profile, progression, economy | JSON file in `Application.persistentDataPath` |
| Trivial preferences (volume, language) | `PlayerPrefs` (only these — nothing else) |
| Large binary assets (replays, screenshots) | Binary file, `Application.persistentDataPath` |
| Cloud sync / leaderboards | Server-side — never store authority locally |

**PlayerPrefs Hard Constraint**: `PlayerPrefs` is stored in plaintext in the OS registry/plist. It must NEVER contain currency values, premium item flags, stats, or progression data. Violation of this rule is a Critical security issue.

---

## Phase 3 — Implementation Patterns

### Tier 1 — Plain JSON (Trivial Data)

```csharp
// SettingsRepository.cs
public sealed class SettingsRepository : ISettingsRepository
{
    private readonly string _filePath = 
        Path.Combine(Application.persistentDataPath, "settings.json");

    public async UniTask SaveAsync(SettingsData data, CancellationToken ct = default)
    {
        string json = JsonUtility.ToJson(data, prettyPrint: false);
        await File.WriteAllTextAsync(_filePath, json, ct);
    }

    public async UniTask<SettingsData> LoadAsync(CancellationToken ct = default)
    {
        if (!File.Exists(_filePath)) return SettingsData.Default;

        string json = await File.ReadAllTextAsync(_filePath, ct);
        return JsonUtility.FromJson<SettingsData>(json);
    }
}
```

### Tier 2 — Integrity Signed JSON (Standard Data)
Attaches an HMAC-SHA256 signature to detect tampering. If the signature doesn't match on load → data is rejected and reset.

```csharp
// SaveRepository.cs
public sealed class SaveRepository : ISaveRepository
{
    // Key must NOT be hardcoded in production — derive from device ID or obfuscated config
    private static readonly byte[] HmacKey = 
        System.Text.Encoding.UTF8.GetBytes("REPLACE_WITH_DERIVED_KEY");

    private readonly string _filePath = 
        Path.Combine(Application.persistentDataPath, "save.json");

    public async UniTask SaveAsync(SaveData data, CancellationToken ct = default)
    {
        string json = JsonUtility.ToJson(data);
        string signature = ComputeHmac(json);

        var envelope = new SaveEnvelope { Payload = json, Signature = signature, Version = SaveData.CurrentVersion };
        string envelopeJson = JsonUtility.ToJson(envelope);

        await File.WriteAllTextAsync(_filePath, envelopeJson, ct);
    }

    public async UniTask<SaveData> LoadAsync(CancellationToken ct = default)
    {
        if (!File.Exists(_filePath)) return SaveData.NewGame();

        string envelopeJson = await File.ReadAllTextAsync(_filePath, ct);
        SaveEnvelope envelope = JsonUtility.FromJson<SaveEnvelope>(envelopeJson);

        // Integrity check — reject tampered saves
        if (ComputeHmac(envelope.Payload) != envelope.Signature)
        {
            Debug.LogWarning("[SaveRepository] Integrity check FAILED. Save data rejected.");
            return SaveData.NewGame(); // Or trigger anti-cheat flow
        }

        SaveData data = JsonUtility.FromJson<SaveData>(envelope.Payload);

        // Version migration
        return MigrateIfNeeded(data, envelope.Version);
    }

    private static string ComputeHmac(string payload)
    {
        using var hmac = new System.Security.Cryptography.HMACSHA256(HmacKey);
        byte[] hash = hmac.ComputeHash(System.Text.Encoding.UTF8.GetBytes(payload));
        return Convert.ToBase64String(hash);
    }

    private static SaveData MigrateIfNeeded(SaveData data, int fromVersion)
    {
        // Version migration chain — add cases as schema evolves
        if (fromVersion < 2) data = MigrateV1ToV2(data);
        if (fromVersion < 3) data = MigrateV2ToV3(data);
        return data;
    }
}

[Serializable]
public sealed class SaveEnvelope
{
    public string Payload;
    public string Signature;
    public int Version;
}
```

### Tier 3 — AES-128 Encrypted + HMAC Signed (Critical Data)
Uses AES-128 encryption with a random IV per save, then signs the ciphertext. Protects both confidentiality and integrity.

```csharp
// CryptoSaveRepository.cs
public sealed class CryptoSaveRepository : ICryptoSaveRepository
{
    // In production: derive key from device-specific data (not hardcoded)
    // e.g., SystemInfo.deviceUniqueIdentifier + server-side pepper
    private static readonly byte[] AesKey = new byte[16]; // 128-bit key — POPULATE SECURELY
    private static readonly byte[] HmacKey = new byte[32]; // 256-bit key — POPULATE SECURELY

    public async UniTask SaveAsync(PlayerData data, CancellationToken ct = default)
    {
        string json = JsonUtility.ToJson(data);
        byte[] plaintext = System.Text.Encoding.UTF8.GetBytes(json);

        // Encrypt with fresh random IV each save
        byte[] (ciphertext, iv) = AesEncrypt(plaintext);

        // Sign the ciphertext (encrypt-then-sign)
        string signature = HmacSign(ciphertext);

        var envelope = new CryptoEnvelope
        {
            Ciphertext = Convert.ToBase64String(ciphertext),
            IV = Convert.ToBase64String(iv),
            Signature = signature,
            Version = PlayerData.CurrentVersion
        };

        string envelopeJson = JsonUtility.ToJson(envelope);
        await File.WriteAllTextAsync(GetPath(), envelopeJson, ct);
    }

    public async UniTask<PlayerData> LoadAsync(CancellationToken ct = default)
    {
        if (!File.Exists(GetPath())) return PlayerData.NewGame();

        string envelopeJson = await File.ReadAllTextAsync(GetPath(), ct);
        CryptoEnvelope envelope = JsonUtility.FromJson<CryptoEnvelope>(envelopeJson);

        byte[] ciphertext = Convert.FromBase64String(envelope.Ciphertext);

        // Verify signature before decrypting (timing-safe compare in prod)
        if (HmacSign(ciphertext) != envelope.Signature)
        {
            Debug.LogError("[CryptoSave] Signature mismatch — possible tampering detected.");
            OnTamperingDetected();
            return PlayerData.NewGame();
        }

        byte[] iv = Convert.FromBase64String(envelope.IV);
        byte[] plaintext = AesDecrypt(ciphertext, iv);
        string json = System.Text.Encoding.UTF8.GetString(plaintext);

        return MigrateIfNeeded(JsonUtility.FromJson<PlayerData>(json), envelope.Version);
    }

    private static (byte[], byte[]) AesEncrypt(byte[] data)
    {
        using var aes = System.Security.Cryptography.Aes.Create();
        aes.Key = AesKey;
        aes.GenerateIV();
        using var encryptor = aes.CreateEncryptor();
        byte[] ciphertext = encryptor.TransformFinalBlock(data, 0, data.Length);
        return (ciphertext, aes.IV);
    }

    private static byte[] AesDecrypt(byte[] ciphertext, byte[] iv)
    {
        using var aes = System.Security.Cryptography.Aes.Create();
        aes.Key = AesKey;
        aes.IV = iv;
        using var decryptor = aes.CreateDecryptor();
        return decryptor.TransformFinalBlock(ciphertext, 0, ciphertext.Length);
    }

    private static string HmacSign(byte[] data)
    {
        using var hmac = new System.Security.Cryptography.HMACSHA256(HmacKey);
        return Convert.ToBase64String(hmac.ComputeHash(data));
    }

    private static void OnTamperingDetected()
    {
        // Options: log analytics event, ban flag, soft reset, notify server
        Debug.LogWarning("[CryptoSave] Anti-cheat: tampering detected.");
    }

    private string GetPath() => Path.Combine(Application.persistentDataPath, "playerdata.dat");
}
```

---

## Phase 4 — Save Versioning & Migration

Every save schema must carry a version number. When the data model evolves, never break old saves — migrate forward.

```csharp
[Serializable]
public sealed class PlayerData
{
    public const int CurrentVersion = 3; // Increment when schema changes

    // v1 fields
    public int Gold;
    public int Level;

    // v2 fields (added sprint 4)
    public string[] UnlockedSkins;

    // v3 fields (added sprint 7)
    public float TotalPlaytimeHours;

    public static PlayerData NewGame() => new() { Gold = 100, Level = 1, UnlockedSkins = Array.Empty<string>() };
}

// Migration functions — one per version gap
private static PlayerData MigrateV1ToV2(PlayerData d) { d.UnlockedSkins = Array.Empty<string>(); return d; }
private static PlayerData MigrateV2ToV3(PlayerData d) { d.TotalPlaytimeHours = 0f; return d; }
```

**Rule**: Never remove a field from `PlayerData` without a matching migration. Rename by adding the new field, migrating the value, then clearing the old one in a subsequent version.

---

## Phase 5 — Async I/O Enforcement

All file operations MUST be async. Synchronous disk I/O blocks the render thread and causes frame hitches during autosaves on mobile.

```csharp
// Autosave pattern — fire-and-forget, safe to call from gameplay
public async UniTaskVoid AutoSaveAsync(CancellationToken ct)
{
    try
    {
        await _saveRepository.SaveAsync(_playerData.Current, ct);
        Debug.Log("[AutoSave] Save complete.");
    }
    catch (OperationCanceledException)
    {
        // App shutting down — expected, ignore
    }
    catch (Exception ex)
    {
        Debug.LogError($"[AutoSave] Failed: {ex.Message}");
    }
}
```

---

## Security Analysis Checklist
When asked to review existing persistence code, verify:
- [ ] No currency / stats stored in `PlayerPrefs`
- [ ] Critical data uses AES encryption + HMAC signature
- [ ] Standard data uses at minimum HMAC integrity check  
- [ ] HMAC key is NOT hardcoded as a plaintext string literal
- [ ] All file I/O is async (no `File.WriteAllText` on main thread)
- [ ] Save schema has a version field with migration support
- [ ] Tamper detection has a defined response (not silently ignored)
- [ ] Old save files from previous schema versions load without crashing

---

## Cross-Skill References
- Integrating saves into feature lifecycle → `skills/create-feature` (Step 3 — Pure Logic)
- Async patterns and CancellationToken usage → `rules/07-async-coroutines.md`
- Testing save/load logic → `skills/test-driven-dev` (EditMode tests, mock file system)
