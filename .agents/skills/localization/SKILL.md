---
name: localization
description: Use when adding multi-language support, setting up Unity Localization package, switching locales at runtime, or handling text/asset localization for mobile release.
---

# Skill: Localization (i18n)

## Capability Overview
The agent designs and implements multi-language support using Unity's official Localization package. When localization is needed, the agent proactively detects hardcoded text, sets up the correct table structure, implements runtime locale switching, and handles platform-specific locale detection — without requiring manual guidance on each step.

---

## Phase 1 — Decision: Do We Need Localization?

Run this check before any implementation:

| Signal | Decision |
|---|---|
| Game targets > 1 language market | ✅ Implement immediately |
| All text is in Inspector string fields | ✅ Must localize — replace with LocalizedString |
| Text is generated at runtime from code | ✅ Must localize — use string tables |
| Game is single-language prototype | ⚠️ Still use LocalizedString — easier to migrate later |
| All "text" is icon-based (no readable strings) | ❌ Localization not needed for text; consider Asset Tables for culture-specific icons |

**Agent proactive trigger**: If any `TextMeshProUGUI.text = "hardcoded string"` is detected in a script, immediately flag it and propose replacing with a LocalizedString reference.

---

## Phase 2 — Setup (Unity Localization Package)

### 2.1 Package Installation
```
Package Manager → Add package by name → com.unity.localization
```

### 2.2 Localization Settings
```
Edit → Project Settings → Localization
→ Create Localization Settings asset
→ Set Default Locale (e.g., English [en])
→ Add supported Locales: Vietnamese [vi], Japanese [ja], Korean [ko], etc.
```

### 2.3 Create String Tables
```
Window → Asset Management → Localization Tables
→ Create New Table Collection → String Table
→ Name: "UI_Strings" (one table per domain — UI, Gameplay, Narrative)
→ Add entries for each locale
```

**Table organization strategy** (agent decides based on content volume):

| Table Name | Contains | Volume |
|---|---|---|
| `UI_Common` | Buttons, labels, generic UI | < 100 entries |
| `UI_HUD` | Health, ammo, score labels | < 50 entries |
| `Gameplay_Prompts` | Tutorial hints, interaction prompts | < 200 entries |
| `Narrative_Dialogue` | Story dialogue, NPC lines | Large — consider Addressables |
| `Meta_StoreStrings` | App Store description, IAP names | < 30 entries |

---

## Phase 3 — Implementation Patterns

### 3.1 Static UI Text — Inspector-Driven (Preferred)
For text that never changes at runtime (button labels, headers):

```csharp
// No code needed — use LocalizeStringEvent component on the TextMeshProUGUI GameObject
// In Inspector:
//   Add Component → Localize String Event
//   Table: UI_Common
//   Key: btn_start_game
//   Target: TextMeshProUGUI.text
```

This is zero-code and automatically updates when locale changes.

### 3.2 Dynamic Text — Code-Driven
For text that changes based on game state (player name, score, item count):

```csharp
// Using Smart Strings for variable substitution
// In Localization Table, Key: "hud_score_label"
// English value: "Score: {score}"
// Vietnamese value: "Điểm: {score}"
// Japanese value: "スコア: {score}"

// In code:
public sealed class ScoreHUDView : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI _scoreText;

    // Reference the localized string asset
    [SerializeField] private LocalizedString _scoreLabel;

    private void UpdateScore(int score)
    {
        // Pass variables into the Smart String
        _scoreLabel.Arguments = new object[] { new { score } };
        _scoreText.text = _scoreLabel.GetLocalizedString();
    }
}
```

### 3.3 Pluralization (Smart Strings)
Handles "1 item" vs "2 items" automatically:

```
// In String Table — Key: "inventory_item_count"
// English: {itemCount:plural:one=# item|other=# items}
// Vietnamese: {itemCount} vật phẩm (Vietnamese has no pluralization)
// Japanese: {itemCount}個のアイテム
```

```csharp
_itemCountLabel.Arguments = new object[] { new { itemCount = _inventory.GetCount() } };
_itemCountLabel.RefreshString(); // Re-evaluates the smart string
```

### 3.4 Localized Assets (Asset Tables)
For assets that differ by locale (locale-specific sprites, audio):

```csharp
// AssetTable entry: "btn_play_sprite"
// Each locale points to a different Sprite asset

[SerializeField] private LocalizedSprite _playButtonSprite;

private void Awake()
{
    _playButtonSprite.LoadAssetAsync().Completed += handle =>
    {
        _playButtonImage.sprite = handle.Result;
    };
}
```

---

## Phase 4 — Runtime Locale Switching

```csharp
// LocaleSelector.cs — registered as a service
public sealed class LocaleSelector : ILocaleSelector
{
    public void SwitchLocale(string localeCode)
    {
        Locale targetLocale = LocalizationSettings.AvailableLocales.GetLocale(localeCode);

        if (targetLocale == null)
        {
            Debug.LogWarning($"[LocaleSelector] Locale not found: {localeCode}");
            return;
        }

        LocalizationSettings.SelectedLocale = targetLocale;
        // All LocalizeStringEvent components auto-update immediately
    }

    public string CurrentLocaleCode => LocalizationSettings.SelectedLocale?.Identifier.Code ?? "en";
}
```

Save the user's locale preference (Tier 1 — trivial data, `PlayerPrefs` is acceptable here):
```csharp
PlayerPrefs.SetString("selected_locale", localeCode);
PlayerPrefs.Save();
```

---

## Phase 5 — Platform Locale Auto-Detection

On first launch, detect the device locale and apply automatically:

```csharp
// PlatformLocaleDetector.cs
public sealed class PlatformLocaleDetector
{
    private static readonly HashSet<string> SupportedLocales = new() { "en", "vi", "ja", "ko", "zh" };

    public string DetectAndApply()
    {
        // Check saved preference first
        string saved = PlayerPrefs.GetString("selected_locale", string.Empty);
        if (!string.IsNullOrEmpty(saved) && SupportedLocales.Contains(saved))
            return saved;

        // Fall back to device system language
        string deviceLocale = Application.systemLanguage switch
        {
            SystemLanguage.Vietnamese => "vi",
            SystemLanguage.Japanese => "ja",
            SystemLanguage.Korean => "ko",
            SystemLanguage.ChineseSimplified => "zh",
            _ => "en"
        };

        // If device locale not supported, fall back to English
        return SupportedLocales.Contains(deviceLocale) ? deviceLocale : "en";
    }
}
```

---

## Phase 6 — RTL Language Support (Arabic, Hebrew)

If supporting RTL languages:
```csharp
// Detect and apply RTL layout direction
private void ApplyTextDirection(string localeCode)
{
    bool isRTL = localeCode is "ar" or "he" or "fa";

    // Flip all HorizontalAlignment in TMP
    foreach (var tmp in FindObjectsOfType<TextMeshProUGUI>())
        tmp.alignment = isRTL ? TextAlignmentOptions.Right : TextAlignmentOptions.Left;

    // Mirror UI layouts
    Canvas.ForceUpdateCanvases();
}
```

---

## Phase 7 — Proactive Hardcoded Text Detection

When reviewing or generating code, the agent automatically flags:

| Pattern | Action |
|---|---|
| `_label.text = "Play"` | Flag: hardcoded string — replace with `LocalizedString` |
| `Debug.Log("some message")` | OK for debug logs — no localization needed |
| `_errorText.text = "Error: " + ex.Message` | Partially flag: key should be localized, exception message can stay |
| `[SerializeField] private string _buttonLabel` | Flag: should be `LocalizedString` type |

---

## Localization Checklist

Before shipping a localized build:
- [ ] All player-visible strings are in a String Table (no hardcoded text in code/Inspector)
- [ ] Smart Strings used for all dynamic content (scores, counts, player name)
- [ ] Platform locale auto-detection tested on a physical device
- [ ] Locale switching tested without app restart
- [ ] All String Table keys follow naming convention: `domain_component_description` (e.g., `ui_btn_start_game`)
- [ ] Localized assets (sprites, audio) load correctly per locale
- [ ] Missing translation fallback verified (falls back to default locale, not blank)
- [ ] String Tables exported and reviewed by native speakers before submission

---

## Cross-Skill References
- Locale preference stored in settings → `skills/data-security-mind` (Tier 1 — PlayerPrefs is acceptable)
- Localized text in UI screens built via MCP → `skills/prompt-to-mcp-builder`
- Runtime locale loading of large narrative tables via Addressables → `skills/asset-loading`
