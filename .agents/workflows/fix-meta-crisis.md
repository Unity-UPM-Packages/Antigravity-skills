# Workflow: Fix Meta Crisis

Emergency recovery for a Unity project with broken references — typically caused by merge conflicts, deleted `.meta` files, or regenerated GUIDs. Symptoms: "Missing Script" errors, pink/magenta materials, null Prefab references in scenes.

---

## Step 0 — STOP — Do Not Open Unity Yet

> ⚠️ **CRITICAL**: If Unity opens and auto-reimports assets, it will regenerate `.meta` files with new GUIDs — making the situation worse. Keep Unity closed until Step 4.

---

## Step 1 — Diagnose the Damage

```bash
# See all changes in the working tree
git status

# Filter for .meta file changes
git status | grep ".meta"

# List assets without a .meta file (untracked assets)
git ls-files --others --exclude-standard Assets/ | grep -v ".meta"

# List .meta files without a corresponding asset (orphaned metas)
git ls-files Assets/ | grep ".meta"
```

Document:
- `.meta` files that were deleted (need restoring)
- New assets that have no `.meta` yet (need proper staging)

---

## Step 2 — Classify Each Missing .meta

For each `.meta` file in the deleted list:

**Case A — Asset still exists, only `.meta` was deleted:**
```bash
# Restore from git — GUID is preserved, no broken references
git restore Assets/Path/To/File.cs.meta
```

**Case B — Both asset and `.meta` were intentionally deleted:**
```bash
# Remove the orphaned .meta from tracking
git rm Assets/Path/To/DeletedFile.cs.meta
```

**Case C — New asset added but `.meta` not yet generated:**
```bash
# Do NOT let Unity auto-create it yet.
# Stage the asset first, then open Unity briefly to generate the .meta, then:
git add Assets/Path/To/NewFile.cs Assets/Path/To/NewFile.cs.meta
```

---

## Step 3 — Resolve YAML Conflicts (if present)

If any `.scene`, `.prefab`, or `.asset` file is in a merge conflict state:

```bash
# Identify conflicted files
git status | grep "both modified"
```

**Never manually edit Unity YAML files.** Use `UnityYAMLMerge`:

```bash
git mergetool  # Automatically invokes UnityYAMLMerge if configured
```

If `UnityYAMLMerge` is not yet configured:
```bash
# Keep your version
git checkout --ours Assets/Scenes/GameScene.unity
git add Assets/Scenes/GameScene.unity

# OR keep the incoming version
git checkout --theirs Assets/Scenes/GameScene.unity
git add Assets/Scenes/GameScene.unity
```

Inform the team which changes were dropped.

---

## Step 4 — Open Unity and Verify

After all `.meta` files are correctly restored:

1. Open Unity Editor
2. Allow the reimport to complete — do **not** interrupt it
3. Check the Console: any "Missing Script" or GUID errors?
4. Open affected Scenes — verify Prefab references are intact
5. Check Materials: any pink/magenta surfaces?

---

## Step 5 — If Missing Script Errors Persist

The GUID has changed. Locate the original:

```bash
# Current GUID in the .meta file
cat Assets/Scripts/MyScript.cs.meta | grep "guid:"

# GUID from the previous commit
git show HEAD~1:Assets/Scripts/MyScript.cs.meta | grep "guid:"
```

If the GUIDs differ, the `.meta` was regenerated. Restore it from history:

```bash
git show HEAD~1:Assets/Scripts/MyScript.cs.meta > Assets/Scripts/MyScript.cs.meta
```

Reopen Unity to reimport with the restored GUID.

---

## Step 6 — Prevent Recurrence

Verify `.gitattributes` exists and is correct:

```bash
cat .gitattributes
```

If missing or incomplete, create/update it:
```
*.unity    merge=unityyamlmerge eol=lf
*.prefab   merge=unityyamlmerge eol=lf
*.asset    merge=unityyamlmerge eol=lf
*.meta     merge=unityyamlmerge eol=lf

*.png      binary
*.jpg      binary
*.fbx      binary
*.mp3      binary
*.wav      binary
*.ogg      binary
```

Configure `UnityYAMLMerge` in `.gitconfig` if not already set (see `skills/git-workflow` Phase 3.3).

---

## Step 7 — Commit the Fix

```bash
git add .
git commit -m "fix(meta): restore broken .meta files after merge conflict"

# If .gitattributes was added or updated
git add .gitattributes
git commit -m "chore: add .gitattributes for Unity YAML merge strategy"
```

---

## Final Verification

- [ ] Unity Console shows 0 errors related to Missing Script or GUID
- [ ] All affected Scenes open without errors
- [ ] All Materials render correctly (no pink)
- [ ] All Prefab instances in the hierarchy have correct components
- [ ] `.gitattributes` is committed at the project root

---

## Cross-Skill References
- Git conflict resolution in detail → `skills/git-workflow` (Phase 3)
- Disaster recovery (reflog, hard reset) → `skills/git-workflow` (Phase 4)
