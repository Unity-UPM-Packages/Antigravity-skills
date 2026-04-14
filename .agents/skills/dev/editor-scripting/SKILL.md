---
name: editor-scripting
description: Use when asked to automate bulk asset modifications, or write Unity Editor Menu tools and Custom Inspectors. Also activates proactively when an MCP operation fails and an Editor Script fallback is needed, or when the AI detects any repetitive manual task (bulk renaming, texture re-compression, missing component fixing) that should be automated with a single menu click.
---

# Skill: Editor Toolsmithing & Automation

## Capability Overview
You are a master of `#if UNITY_EDITOR` and `UnityEditor` namespace programming. Rather than providing long instructions telling the user to click through the Inspector, proactively write Custom Editors, `MenuItem` tools, and `AssetPostprocessor` hooks to automate tedious processes. **One click should replace any repetitive manual task.**

---

## Core Principles

### 1. Bulk Orchestration
If a task requires modifying multiple prefabs, compressing textures, renaming assets, or reorganizing scenes — NEVER ask the user to do it manually. Generate an Editor script under `Editor/Tools/` that processes assets programmatically.

```csharp
// Editor/Tools/TextureCompressionTool.cs
using UnityEditor;
using UnityEngine;

public static class TextureCompressionTool
{
    [MenuItem("Tools/Antigravity/Compress All UI Sprites")]
    public static void CompressAllUISprites()
    {
        string[] guids = AssetDatabase.FindAssets("t:Texture2D", new[] { "Assets/Art/UI" });

        foreach (string guid in guids)
        {
            string path = AssetDatabase.GUIDToAssetPath(guid);
            TextureImporter importer = AssetImporter.GetAtPath(path) as TextureImporter;

            if (importer == null) continue;

            importer.textureCompression = TextureImporterCompression.Compressed;
            importer.crunchedCompression = true;
            importer.compressionQuality = 75;
            importer.SaveAndReimport();
        }

        Debug.Log($"[TextureCompressionTool] Processed {guids.Length} textures.");
    }
}
```

### 2. Custom Inspector with SerializedObject
For complex MonoBehaviour components, write a Custom Inspector to improve data entry UX and add runtime debugging buttons.

```csharp
// Editor/Inspectors/HealthComponentEditor.cs
using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(HealthComponent))]
public sealed class HealthComponentEditor : Editor
{
    private SerializedProperty _maxHealth;
    private SerializedProperty _currentHealth;

    private void OnEnable()
    {
        // Always use SerializedProperty — never cast target directly for field access
        _maxHealth = serializedObject.FindProperty("_maxHealth");
        _currentHealth = serializedObject.FindProperty("_currentHealth");
    }

    public override void OnInspectorGUI()
    {
        serializedObject.Update(); // Sync serialized data

        EditorGUILayout.LabelField("Health Configuration", EditorStyles.boldLabel);
        EditorGUILayout.PropertyField(_maxHealth);

        // Runtime-only debug panel
        if (Application.isPlaying)
        {
            EditorGUILayout.Space();
            EditorGUILayout.LabelField("Runtime Debug", EditorStyles.boldLabel);
            EditorGUILayout.PropertyField(_currentHealth);

            if (GUILayout.Button("Kill (Debug)"))
                ((HealthComponent)target).TakeDamage(9999);
        }

        serializedObject.ApplyModifiedProperties(); // Commit changes with Undo support
    }
}
```

**Critical Rule**: Always use `SerializedProperty` + `serializedObject.Update/ApplyModifiedProperties` for Custom Inspectors. Never directly modify target fields via casting — this breaks Undo/Redo and Prefab Override tracking.

### 3. AssetPostprocessor Hooks
Automate import pipeline configuration when new assets are added to specific folders.

```csharp
// Editor/Pipeline/SpriteImportConfig.cs
using UnityEditor;

public sealed class SpriteImportConfig : AssetPostprocessor
{
    private void OnPreprocessTexture()
    {
        if (!assetPath.StartsWith("Assets/Art/UI/")) return;

        TextureImporter importer = (TextureImporter)assetImporter;
        importer.textureType = TextureImporterType.Sprite;
        importer.spritePixelsPerUnit = 100;
        importer.filterMode = FilterMode.Bilinear;
        importer.mipmapEnabled = false; // Mobile: disable mipmaps for UI
    }
}
```

### 4. MCP Fallback Pivot
If the Unity MCP Server fails to configure a complex serialized structure (e.g., nested prefab component arrays, ScriptableObject graph linking), immediately fall back to writing an ephemeral Editor Script that completes the setup in **1 human click**.

Pattern for MCP fallback tools:
```csharp
[MenuItem("Tools/Antigravity/MCP Fallback — Wire Enemy Config")]
public static void WireEnemyConfig()
{
    // Find all EnemyController prefabs and wire their EnemyConfigSO reference
    string[] guids = AssetDatabase.FindAssets("t:Prefab", new[] { "Assets/Prefabs/Enemies" });
    EnemyConfigSO config = AssetDatabase.LoadAssetAtPath<EnemyConfigSO>("Assets/Configs/DefaultEnemyConfig.asset");

    foreach (string guid in guids)
    {
        string path = AssetDatabase.GUIDToAssetPath(guid);
        GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(path);
        EnemyController ec = prefab.GetComponent<EnemyController>();

        if (ec != null)
        {
            SerializedObject so = new SerializedObject(ec);
            so.FindProperty("_config").objectReferenceValue = config;
            so.ApplyModifiedProperties();
            PrefabUtility.SavePrefabAsset(prefab);
        }
    }

    AssetDatabase.SaveAssets();
    Debug.Log("[MCP Fallback] Enemy config wiring complete.");
}
```

### 5. Build Safety — Mandatory Wrapper
**All Editor code MUST be inside `Editor/` folder or wrapped in preprocessor directives.** Failure to do this causes Android/iOS production builds to fail with compile errors.

```csharp
// Option A: Place file in any Editor/ subfolder (preferred)
// Assets/Editor/Tools/MyTool.cs

// Option B: Preprocessor guard for mixed files
#if UNITY_EDITOR
using UnityEditor;
// ... editor code
#endif
```

---

## Common Automation Recipes

| Task | Tool Type | Trigger |
|---|---|---|
| Re-import all audio with mobile settings | `MenuItem` + `AssetImporter` loop | Manual click |
| Auto-assign Layer to prefabs by folder | `AssetPostprocessor.OnPostprocessPrefab` | On import |
| Batch rename assets by convention | `MenuItem` + `AssetDatabase.RenameAsset` | Manual click |
| Validate scene setup before build | `IPreprocessBuildWithReport` | On build |
| Generate ScriptableObject entries from CSV | `MenuItem` + `ScriptableObject.CreateInstance` | Manual click |

---

## Cross-Skill References
- When MCP is available and stable → `skills/mcp-auto-architect`
- For build-time validation hooks → `rules/06-git-conventions.md` (pre-commit checklist)
- For connecting Editor tools to Addressables pipeline → `skills/asset-loading`
