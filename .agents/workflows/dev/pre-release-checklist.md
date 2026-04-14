---
description: Ten-step pre-release checklist covering build config, asset compression, security audit, store requirements, and release tagging.
---

# Workflow: Pre-Release Checklist

A comprehensive checklist to complete before submitting to the App Store or Google Play. Execute in order — do not skip any step.

---

## Step 1 — Version & Bundle Identifier

Open **Edit → Project Settings → Player**:
- [ ] `Bundle Identifier` uses correct format: `com.companyname.gamename`
- [ ] `Version` incremented from last release (semantic versioning: `1.2.3`)
- [ ] `Bundle Version Code` (Android) / `Build Number` (iOS) incremented by 1
- [ ] `Product Name` contains no special characters
- [ ] `Company Name` matches the store developer account

---

## Step 2 — Build Settings

Open **File → Build Settings**:
- [ ] Correct platform selected (Android / iOS)
- [ ] Only necessary scenes present in the Build Scenes list
- [ ] First scene (index 0) is the Boot or Splash scene
- [ ] `Development Build` = **OFF**
- [ ] `Script Debugging` = **OFF**

---

## Step 3 — Player Settings (Android)

**Edit → Project Settings → Player → Android**:
- [ ] `Minimum API Level` appropriate for target audience (API 28+ recommended)
- [ ] `Target API Level` = latest stable
- [ ] `Scripting Backend` = **IL2CPP** (not Mono)
- [ ] `Target Architectures` = **ARM64** ✅ (+ ARMv7 if legacy devices needed)
- [ ] `Internet Access` = Required (if the game makes network calls)
- [ ] Keystore configured with correct password
- [ ] `Managed Stripping Level` = Medium or High

**Edit → Project Settings → Player → iOS**:
- [ ] `Target minimum iOS Version` = 14.0 or above
- [ ] `Architecture` = ARM64
- [ ] `Camera Usage Description` filled in (if camera is used)
- [ ] `Microphone Usage Description` filled in (if microphone is used)

---

## Step 4 — Icons & Splash Screen

- [ ] App icon set for all required sizes on both platforms
- [ ] Default Unity icon ("Made with Unity") is NOT used
- [ ] Splash screen background color matches the game's brand
- [ ] Splash screen duration does not exceed 2 seconds

---

## Step 5 — Quality & Graphics Settings

**Edit → Project Settings → Quality**:
- [ ] Production build uses the correct Quality Level (not "Ultra")
- [ ] `vSync Count` = Don't Sync (use `Application.targetFrameRate` instead)
- [ ] `Shadow Distance` is appropriate (or disabled for 2D mobile games)

**Edit → Project Settings → Graphics**:
- [ ] Active URP Asset is the Mobile-tuned one (not the PC/High-Fidelity URP)
- [ ] `Shader Stripping` enabled to reduce build size

---

## Step 6 — Quick Performance Check

Run `workflows/performance-audit` if it has not been executed this sprint.

Quick final verification:
- [ ] Target frame rate explicitly set: `Application.targetFrameRate = 60` (or 30 for low-end targets)
- [ ] No `Debug.Log` calls in production paths (use `#if UNITY_EDITOR` guard):

```bash
grep -rn "Debug\.Log" Assets/Scripts --include="*.cs" | grep -v "//" | grep -v "UNITY_EDITOR"
```

---

## Step 7 — Audio & Localization

- [ ] All `AudioClip` assets use appropriate compression for their category
- [ ] Background music: `Load Type = Streaming`
- [ ] Short SFX (< 5s): `Load Type = Decompress On Load`
- [ ] Audio pauses when the app backgrounds: `AudioListener.pause = true`
- [ ] If localized: all locales tested, no empty strings

---

## Step 8 — Final Build Test on Device

```
1. Produce a Release build (Development Build = OFF)
2. Install on a physical device (not an emulator or simulator)
3. Cold start test: app launches from scratch without crashing
4. Full session test: gameplay → pause → resume → quit
5. Airplane mode test: no unhandled network exceptions
6. Memory monitor: stays under 1.5 GB on the lowest supported device
```

---

## Step 9 — Store Metadata Checklist

- [ ] App title (≤ 30 chars on iOS, ≤ 50 chars on Android)
- [ ] Short description (≤ 80 chars)
- [ ] Full description free of prohibited keywords
- [ ] Screenshots in correct dimensions for each device category
- [ ] App preview video (optional, but improves conversion rate)
- [ ] Content Rating questionnaire completed
- [ ] Privacy Policy URL valid and accessible
- [ ] Age Rating appropriate for game content

---

## Step 10 — Tag the Release

```bash
git add .
git commit -m "chore: bump version to x.y.z for release build"
git tag -a "v1.0.0" -m "Release v1.0.0 — [brief description]"
git push origin main --tags
```

---

## Cross-Skill References
- Pre-ship performance issues → `workflows/performance-audit`
- Git tagging conventions → `skills/git-workflow`
- Save data security final check → `skills/data-security-mind`
