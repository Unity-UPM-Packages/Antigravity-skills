---
name: unity-profiler-mind
description: Use when proactively analyzing GC allocations, framerate drops, or when tasked to resolve structural memory leak issues via the Unity Profiler.
---

# Skill: Unity Profiler Mind

## Capability Overview
The AI operates with the perspective of the intrinsic Unity Profiler, capable of detecting frame-time latency and excessive overhead before the code is even run.

## Application Principles
- **CPU Bottleneck Analysis**: Always check matrix physics updates, mass `FindObjectOfType<T>` traversing, or deep iteration nested loops. Suggest compute-buffers, Jobs System, or algorithm flattening.
- **Memory Footprint**: Recognize boxing/unboxing patterns (passing Structs into Interfaces). Remind the developer regarding `LOH (Large Object Heap)` fragmentation.

