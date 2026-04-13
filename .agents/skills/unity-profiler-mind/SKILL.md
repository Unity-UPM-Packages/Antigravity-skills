---
name: unity-profiler-mind
description: Use when investigating framerate drops, GC allocation spikes, memory leaks, or rendering bottlenecks in Unity. Also activates proactively when the AI detects new allocations in Update/FixedUpdate loops, FindObjectOfType calls, string concatenation in hot paths, or unbounded collections during any code task.
---

# Skill: Unity Profiler Mind

## Capability Overview
The AI operates with the perspective of an embedded Unity Profiler, capable of detecting frame-time latency and excessive overhead **before the code runs**. When performance concerns arise, this skill provides a structured investigation protocol rather than guesswork.

---

## Investigation Protocol

### Step 1 — Classify the Symptom
Identify which performance domain is affected before diving in:

| Symptom | Likely Domain | Tool to Use |
|---|---|---|
| Hitches/stutters every N seconds | GC Allocation spike | CPU Profiler + Memory Profiler |
| Consistent low FPS (not spikes) | CPU bottleneck or GPU overdraw | CPU Profiler + Frame Debugger |
| Memory grows over time (never freed) | Memory leak (asset or object) | Memory Profiler (snapshot comparison) |
| First frame hitch after scene load | Asset loading synchronous block | CPU Profiler + Addressables Profiler |
| UI lag when opening panels | Canvas rebuild cascade | UI Profiler + Frame Debugger |
| Battery drain / thermal throttle | GPU overdraw or excessive CPU wakeups | Frame Debugger + GPU Profiler |

---

## CPU Profiler Analysis

### Common Hot-Path Anti-Patterns
Scan for these signals in method self-time rankings:

| Signal | Root Cause | Fix |
|---|---|---|
| `GC.Collect` appears in timeline | Allocation on hot path | Eliminate `new`, cache, pool |
| `Physics.Simulate` or `FixedUpdate` spikes | Too many Rigidbody queries / colliders | Reduce collider count, use layers |
| `FindObjectOfType<T>` in non-init paths | Called inside Update or event handlers | Cache result in `Awake` |
| Deep nested loops in `Update` | O(n²) per-frame iteration | Flatten data, use spatial hashing |
| `Canvas.BuildBatch` / `Canvas.SendWillRenderCanvases` | Canvas rebuild cascade | Isolate dynamic elements per Canvas |
| `Camera.Render` occupying > 8ms | Overdraw / heavy shader | Frame Debugger for overdraw map |
| `Animator.Update` taking excessive time | Too many live animators | Disable animators on off-screen objects |

### C# Jobs System Migration Path
For any CPU-intensive per-frame loop (pathfinding, NPC position updates, physics sampling), migrate to C# Job System:

```csharp
// ❌ Main thread bottleneck
private void Update()
{
    for (int i = 0; i < _enemies.Count; i++)
        _enemies[i].UpdatePathfinding(); // Blocks main thread
}

// ✅ Burst-compiled parallel job
[BurstCompile]
public struct PathfindingJob : IJobParallelFor
{
    public NativeArray<float3> Positions;
    public NativeArray<float3> Targets;
    public NativeArray<float3> Results;

    public void Execute(int index)
    {
        // Pure math — no managed object access
        Results[index] = math.normalize(Targets[index] - Positions[index]);
    }
}

// Schedule from MonoBehaviour
private JobHandle _jobHandle;
private void Update()
{
    _jobHandle = new PathfindingJob { ... }.Schedule(_enemies.Count, 64);
}
private void LateUpdate() => _jobHandle.Complete(); // Sync at end of frame
```

---

## Memory Profiler Workflow

### Snapshot Comparison Process
1. **Take Baseline Snapshot** at game start (clean state)
2. **Perform the suspect action** (open/close inventory 5 times, load/unload scene 3 times)
3. **Take Second Snapshot** after actions
4. In Memory Profiler: **Compare Snapshots** → filter by "New Objects" or "Delta"
5. Identify object types that grew — these are candidates for leaks

### Common Leak Patterns
| Pattern | Cause | Fix |
|---|---|---|
| `Texture2D` count grows | Assets loaded but `Addressables.Release()` not called | Pair every load with release in `OnDestroy` |
| `Action` / `EventHandler` count grows | Event subscriptions not unsubscribed | Unsubscribe in `OnDisable` / `Dispose` |
| `GameObject` count grows | `Instantiate` without `Destroy` / pool return | Object Pool enforcement |
| `string` allocation grows | String concatenation in hot paths | `StringBuilder` or `TMP_Text.SetText` overload |
| Boxing allocations | Struct passed as `object` or `interface` | Use generic constraints `<T> where T : struct` |

### LOH (Large Object Heap) Fragmentation
Any managed allocation > 85 KB goes directly to the LOH. LOH is never compacted, causing permanent fragmentation:
- Avoid large managed arrays that shrink/grow — use `NativeArray<T>` from Unity Collections
- Large `Texture2D` created in code → use GPU-resident textures via `AsyncGPUReadback` instead
- Pre-size `List<T>` with expected capacity: `new List<ItemData>(capacity: 100)` avoids resize allocations

---

## Frame Debugger — GPU Overdraw Analysis

Use the Frame Debugger (`Window > Analysis > Frame Debugger`) to:
1. Enable **Overdraw** mode to visualize layers of transparent rendering
2. Hot spots appear bright — every transparent layer costs additional GPU fill rate
3. Fix strategies:
   - Use opaque materials where possible
   - Merge overlapping UI layers into a single sprite atlas
   - Disable `Bloom` / `Distortion` effects on low-end mobile tiers

---

## Pre-Shipping Performance Checklist
Before any build submission or milestone review, verify:

- [ ] No `GC.Alloc` > 0 bytes in any method called during gameplay (outside init)
- [ ] `Update()` methods collectively take < 4ms on target device
- [ ] Memory Profiler snapshot shows no growing object types after 5 full game loops
- [ ] Frame Debugger shows < 3 overdraw layers on average across all screens
- [ ] No `FindObjectOfType` calls outside `Awake/Start`
- [ ] All dynamic particle FX using `ParticleSystem.Stop` on return-to-pool
- [ ] Addressables handles released for all dynamically loaded assets

---

## Cross-Skill References
- Canvas rebuild profiling → `skills/ui-performance` (Canvas Partitioning section)
- Asset memory leak from unreleased handles → `skills/asset-loading` (Memory Release section)
- Migrating hot paths to Jobs System → Unity.Jobs + Burst package documentation
- Identifying test regressions via performance → `skills/test-driven-dev`
