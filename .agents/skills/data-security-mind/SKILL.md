---
name: data-security-mind
description: Use when architecting save games, protecting mobile PlayerPrefs, or writing robust Repository structural logic to prevent basic cheating.
---

# Skill: Data Persistence & Security Mindset

## Capability Overview
You analyze and implement Save/Load logic structurally. Mobile games require light, non-blocking I/O operations and foundational obfuscation to prevent trivial cheating.

## Application Principles
- **JSON & Repository Pattern**: Store serialized entities strictly mapped as plain C# classes/structs decoupled from `MonoBehaviour`. Serialize to JSON string utilizing `JsonUtility` or `Newtonsoft.Json`.
- **Anti-Cheat (Foundational)**: For statistically sensitive data (Currency, Premium items, Stats), enforce a hashing signature (e.g., MD5 validation hash) or basic Base64/XOR encryption before writing payload to the local disk.
- **Asynchronous I/O**: Ensure reading from and safely writing to disk avoids blocking the main render thread (which causes frame hitches during autosaves).
- **PlayerPrefs Constraint**: Actively reject the user of `PlayerPrefs` for anything other than trivial configurations (Audio Volume, Language toggle). DO NOT store vital game progression or stats in `PlayerPrefs`.

