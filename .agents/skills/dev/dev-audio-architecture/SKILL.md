---
name: dev-audio-architecture
description: Use when designing audio systems, managing music transitions, pooling audio sources, or optimizing mobile audio performance in Unity.
---

# Skill: Audio Architecture

---

## Core Architecture: AudioService

The audio system is registered as a singleton service injected via DI. **No system should call `AudioSource.Play()` directly** — all audio requests go through the `IAudioService` interface.

```csharp
// IAudioService.cs — in Game.Core.asmdef
public interface IAudioService
{
    void PlaySFX(AudioClipKey key, Vector3? worldPosition = null);
    void PlayMusic(AudioClipKey key, float fadeDuration = 1f);
    void StopMusic(float fadeDuration = 1f);
    void SetVolume(AudioChannel channel, float normalizedVolume);
    void SetMuted(AudioChannel channel, bool muted);
}

public enum AudioClipKey
{
    // SFX
    UI_ButtonClick,
    UI_PanelOpen,
    Player_Jump,
    Player_Land,
    Player_Die,
    Enemy_Hurt,
    Enemy_Die,
    Weapon_Fire,
    Weapon_Reload,
    // Music
    Music_MainMenu,
    Music_Gameplay,
    Music_Victory,
    Music_GameOver,
}

public enum AudioChannel { Master, Music, SFX, UI }
```

---

## AudioService Implementation

### AudioSource Pool
Never `Instantiate` or `Destroy` AudioSources during gameplay. Use a fixed-size pool:

```csharp
// AudioService.cs — in Game.Gameplay.asmdef
public sealed class AudioService : IAudioService
{
    private readonly AudioSource _musicSource;
    private readonly Queue<AudioSource> _sfxPool;
    private readonly AudioClipRegistry _registry;
    private readonly AudioMixer _mixer;

    private const int SfxPoolSize = 8; // Tune per game complexity

    public AudioService(AudioClipRegistry registry, AudioMixer mixer, Transform audioRoot)
    {
        _registry = registry;
        _mixer = mixer;

        _musicSource = CreateAudioSource(audioRoot, "MusicSource");
        _musicSource.loop = true;
        _musicSource.outputAudioMixerGroup = mixer.FindMatchingGroups("Music")[0];

        _sfxPool = new Queue<AudioSource>(SfxPoolSize);
        for (int i = 0; i < SfxPoolSize; i++)
        {
            var source = CreateAudioSource(audioRoot, $"SFXSource_{i}");
            source.outputAudioMixerGroup = mixer.FindMatchingGroups("SFX")[0];
            _sfxPool.Enqueue(source);
        }
    }

    public void PlaySFX(AudioClipKey key, Vector3? worldPosition = null)
    {
        if (!_sfxPool.TryDequeue(out AudioSource source))
        {
            Debug.LogWarning("[AudioService] SFX pool exhausted. Increase SfxPoolSize.");
            return;
        }

        AudioClip clip = _registry.GetClip(key);
        source.clip = clip;

        if (worldPosition.HasValue)
            source.transform.position = worldPosition.Value;

        source.Play();

        ReturnToPoolAfter(source, clip.length).Forget();
    }

    public async void PlayMusic(AudioClipKey key, float fadeDuration = 1f)
    {
        await FadeOut(_musicSource, fadeDuration);
        _musicSource.clip = _registry.GetClip(key);
        _musicSource.Play();
        await FadeIn(_musicSource, fadeDuration);
    }

    public void SetVolume(AudioChannel channel, float normalizedVolume)
    {
        // AudioMixer uses logarithmic dB scale
        float db = normalizedVolume > 0 ? Mathf.Log10(normalizedVolume) * 20f : -80f;
        _mixer.SetFloat(channel.ToString(), db);
    }

    private async UniTaskVoid ReturnToPoolAfter(AudioSource source, float delay)
    {
        await UniTask.Delay(TimeSpan.FromSeconds(delay));
        source.clip = null;
        _sfxPool.Enqueue(source);
    }

    private static AudioSource CreateAudioSource(Transform parent, string name)
    {
        var go = new GameObject(name);
        go.transform.SetParent(parent);
        return go.AddComponent<AudioSource>();
    }
}
```

---

## AudioClip Registry (ScriptableObject)

```csharp
// AudioClipRegistry.cs — ScriptableObject
[CreateAssetMenu(fileName = "AudioClipRegistry", menuName = "Antigravity/Audio/Clip Registry")]
public sealed class AudioClipRegistry : ScriptableObject
{
    [Serializable]
    public struct ClipEntry
    {
        public AudioClipKey Key;
        public AudioClip Clip;
        [Range(0f, 1f)] public float Volume;
        [Range(0.9f, 1.1f)] public float PitchVariance; // Randomize for organic feel
    }

    [SerializeField] private ClipEntry[] _entries;
    private Dictionary<AudioClipKey, ClipEntry> _lookup;

    private void OnEnable()
    {
        _lookup = new Dictionary<AudioClipKey, ClipEntry>(_entries.Length);
        foreach (var entry in _entries)
            _lookup[entry.Key] = entry;
    }

    public AudioClip GetClip(AudioClipKey key)
    {
        if (_lookup.TryGetValue(key, out ClipEntry entry))
            return entry.Clip;

        Debug.LogError($"[AudioRegistry] Missing clip for key: {key}");
        return null;
    }
}
```

---

## AudioMixer Structure

```
AudioMixer (Master)
├── Music Group
│   └── Volume exposed: "Music"
├── SFX Group
│   └── Volume exposed: "SFX"
└── UI Group
    └── Volume exposed: "UI"
```

Link volume parameter names to `AudioChannel` enum values for clean `SetFloat` calls.

**Ducking**: Apply a `Duck Volume` effect on Music group, triggered by SFX group's send level. This automatically lowers music volume during intense SFX moments.

---

## Mobile Audio Optimization

| Setting | Mobile Recommendation | Reason |
|---|---|---|
| Compression Format | **Vorbis** (Android) / **AAC** (iOS) | Best compression ratio for mobile |
| Sample Rate | 22050 Hz for SFX, 44100 Hz for Music | 22050 halves memory without audible loss |
| Load Type | `Compressed In Memory` for SFX | Fast decode, low memory |
| Load Type | `Streaming` for long music tracks | Avoids loading full track into RAM |
| Force Mono | ✅ for SFX | Halves SFX memory |
| Preload Audio Data | ✅ for critical SFX | Eliminates first-play delay |

---

## Audio Event Flow

Game systems must **never** hold a reference to `IAudioService` unless they genuinely need to trigger audio. Prefer event-driven audio requests:

```csharp
// PlayerController fires a domain event — it knows nothing about audio
_health.OnDied += () => OnPlayerDied?.Invoke();

// AudioListener bridges the domain event to the audio service
// Receives dependencies via Construct() — no VContainer [Inject] needed
public sealed class PlayerAudioListener : MonoBehaviour
{
    private IAudioService _audio;
    private IPlayerEvents _player;

    /// <summary>Injects audio service and player event source. Call from the scene Bootstrapper.</summary>
    public void Construct(IAudioService audio, IPlayerEvents player)
    {
        _audio = audio;
        _player = player;
    }

    private void OnEnable()  => _player.OnPlayerDied += HandlePlayerDied;
    private void OnDisable() => _player.OnPlayerDied -= HandlePlayerDied;

    private void HandlePlayerDied() => _audio.PlaySFX(AudioClipKey.Player_Die);
}
```

This keeps the audio system fully decoupled from game logic.

---

## Cross-Skill References
- AudioService registered as persistent singleton → `skills/scene-management` (DontDestroyOnLoad section)
- AudioClip assets loaded via Addressables for live-ops music → `skills/asset-loading`
- Wiring AudioListener components → `skills/modular-design` (Event-Driven section)
