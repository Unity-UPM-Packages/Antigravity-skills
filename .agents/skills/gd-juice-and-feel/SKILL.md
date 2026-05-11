---
name: gd-juice-and-feel
description: "[Game Designer] Use when designing, speccing, or auditing any sensory experience — reactive game feel (input feedback), juice language (motion, effects, emotion), ambient life, and information delivery."
---

# Skill: Juice & Feel

## When to use this skill
- User asks about "game feel", "juice", "polish", "it doesn't feel alive", "it feels flat"
- Designing how an action, event, reward, or notification should feel
- Describing motion, animation, or effect behavior to other roles
- Auditing any feature where the sensory experience feels weak or missing
- Writing cross-team specs for Dev, 2D, or 3D covering motion and effects

---

# PART 1: Reactive Feel (Input-Driven)

Game feel as a response to player input: every action must close the loop.

```
Player Input
    ↓ (Responsiveness — is there lag? delay?)
Game Action
    ↓ (Weight — does it feel heavy/light/impactful?)
Feedback Burst
    ↓ (Juice — visual + audio + haptic explosion)
Consequence
    ↓ (Meaning — does it matter?)
```

## 1.1 Input Responsiveness

| Parameter | Recommended (mobile) | Notes |
|---|---|---|
| Input lag | < 50ms | > 100ms feels "floaty" |
| Jump buffer | 80–120ms | Accept jump input slightly before ground |
| Coyote time | 80–150ms | Allow jump after walking off edge |
| Input queue | 1 frame buffer | Prevents dropped inputs on fast taps |
| Dead zone | 0.15–0.25 (analog) | Ignore micro-movements |

## 1.2 Animation Timing

| Phase | Principle | Timing |
|---|---|---|
| **Anticipation** | Wind-up before action | 4–8 frames |
| **Action** | The actual impact moment | 2–6 frames (fastest part) |
| **Follow-through** | Overshoot + settle | 6–12 frames |
| **Recovery** | Return to idle | 4–8 frames |

### Hit-Stop (Freeze Frame)
| Impact type | Hit-stop | Scale punch |
|---|---|---|
| Light hit | 2–3 frames | 1.05× |
| Medium hit | 4–5 frames | 1.10× |
| Heavy hit / KO | 6–8 frames | 1.15× |
| Ultimate skill | 10–15 frames + slow-mo | 0.3× time scale |

## 1.3 Reactive Juice Layers

| Layer | Tool | Role |
|---|---|---|
| Screen Shake | Camera position offset | 🛠 Dev |
| Hit Flash | Sprite/model flash white | 🛠 Dev |
| Particle Burst | VFX on impact point | 🎲 3D |
| Screen Flash | Full-screen color overlay | 🛠 Dev / 🎨 2D |
| UI Number Pop | Damage/score number spray | 🎨 2D |
| Sound Effect | Impact / collect / UI sound | 🛠 Dev |
| Haptic Feedback | Phone vibrate / rumble | 🛠 Dev |
| Slow Motion | Time scale 0.2–0.5× for 0.1–0.3s | 🛠 Dev |

### Juice Budget per Action Type
| Action | Layers recommended |
|---|---|
| Light tap / collect | 2–3 layers |
| Normal attack | 3–4 layers |
| Heavy / special attack | 5–6 layers |
| Boss death / victory | All layers |
| UI button tap | 1–2 layers (subtle) |

## 1.4 Camera Feel

| Parameter | Value | Notes |
|---|---|---|
| Follow smoothing | 0.1–0.2 lerp | Laggy camera = spatial disconnect |
| Lookahead | 0.5–1.5 units | Camera leads slightly ahead of movement |
| FOV punch | +5–10° for 0.1s | On big impacts or speed bursts |
| Shake magnitude | 0.1–0.5 units | Scale to impact weight |
| Shake duration | 0.15–0.4s | Decays exponentially |

---

# PART 2: Juice Language (Event & Ambient-Driven)

**Game Juice = The emotional language of your game.**
Everything the player sees, hears, or feels — not to make gameplay work, but to make the brain feel satisfied, excited, or informed.

This part covers effects and motion that are NOT triggered by direct player input: game events, rewards, notifications, transitions, and atmospheric feel.

---

## 2.1 The Six Purposes of Juice

Every effect must serve at least one purpose:

| Purpose | Signal | Example |
|---|---|---|
| **Notify** | "Something happened" | Quest complete badge, level-up text |
| **Amplify** | "This is IMPORTANT!" | Confetti burst, screen flash on rare drop |
| **Reward** | "You earned something" | Coins flying into wallet, XP bar filling |
| **Breathe Life** | "This world is alive" | Idle character sway, background particle drift |
| **Guide** | "Look here" | Pulsing arrow, glow on next action |
| **Emotion** | "Feel this moment" | Victory fanfare + slow-mo + fireworks |

---

## 2.2 Motion Language

The rules of HOW things move. Use easing curves to convey personality:

| Easing Type | Feel | When to use | Example |
|---|---|---|---|
| **Linear** | Robotic, mechanical | Timers, loading bars | Health bar drain |
| **Ease In** | Starts slow, accelerates | Objects leaving screen | Panel sliding out |
| **Ease Out** | Starts fast, decelerates | Objects arriving | Panel sliding in |
| **Ease In-Out** | Smooth and natural | Most UI transitions | Modal dialog open |
| **Bounce** | Playful, energetic | Reward appearances | Chest lid popping open |
| **Elastic** | Springy, alive | Attention-grabbing | Notification badge pop |
| **Back** | Overshoot then settle | Confident, snappy | Button press scale-down |

### Motion Vocabulary
| Property | Spec format | Example |
|---|---|---|
| **Scale** | From → To, Duration, Easing | `0.5 → 1.0, 0.3s, Elastic` |
| **Fade** | Alpha From → To, Duration | `0 → 1, 0.2s, Ease Out` |
| **Slide** | Direction, Distance, Duration | `Up 40px, 0.25s, Ease Out` |
| **Rotation** | Degrees, Duration, Easing | `0 → 360°, 0.4s, Linear` |
| **Color** | From → To Hex, Duration | `#FF0000 → #FFFFFF, 0.1s` |

---

## 2.3 Event-Driven Sequences

Effects triggered by game events (not player input). Can be single or chained:

### Sequence Spec Template
```markdown
## Effect Sequence: [Event Name]
Trigger: [Game event — e.g., "Player reaches Level 10"]

| Step | Delay | Effect | Duration | Spec |
|---|---|---|---|---|
| 1 | 0ms | Screen flash | 0.15s | White, fade out |
| 2 | 50ms | "LEVEL UP!" text | — | Scale 0→1.2→1.0, Elastic, 0.4s |
| 3 | 100ms | Confetti burst | 2.0s | 50 particles, rainbow, spread 360° |
| 4 | 200ms | XP bar fill | 0.8s | Left→right, Ease Out, glow pulse |
| 5 | 500ms | Reward panel slide in | 0.3s | Bottom→center, Ease Out |
| 6 | 800ms | Sound: fanfare | — | Timing aligns with step 2 |
```

### Common Event Templates

**Reward / Achievement:**
> Text scales in (Elastic) → particle burst → reward icon bounces in → background glow pulses → SFX fanfare

**Notification / Alert:**
> Icon slides in from top-right (Ease Out) → shakes once (Back easing) → badge count bounces (Elastic) → auto slides out after 3s

**Countdown / Urgency:**
> Number ticks down → color shifts red as time decreases → shake intensifies → final flash on zero

**Empty State / Fail:**
> Element fades in (Ease Out) → subtle idle float animation → CTA button pulses gently to guide player

---

## 2.4 Ambient Life

Effects that run continuously to make the world feel alive:

| Layer | Description | Role |
|---|---|---|
| **Character idle** | Subtle breathing, weight shift, blink | 🎲 3D / 🎨 2D |
| **UI idle motion** | Gentle float/pulse on key elements | 🎨 2D |
| **Ambient particles** | Dust motes, embers, snow, sparkle drift | 🎲 3D |
| **Background parallax** | Slow layer movement on menu/scene | 🎨 2D / 🛠 Dev |
| **Environmental audio** | Wind, crowd, nature ambience | 🛠 Dev |
| **Lighting breathe** | Subtle intensity pulse on torches, screens | 🎲 3D |

**Rule**: Ambient motion must be **subtle and non-distracting** — it should be felt, not noticed.

---

## 2.5 Juice Audit Checklist

### Reactive Feel
- [ ] Input lag < 50ms?
- [ ] Impact has hit-stop?
- [ ] At least 2 feedback layers on major actions?
- [ ] Camera responds to impactful moments?

### Juice Language
- [ ] Every game event has a defined effect sequence?
- [ ] Motion uses appropriate easing (not linear for organic things)?
- [ ] Key notifications/rewards unmissable without being annoying?
- [ ] Scene has at least 1 layer of ambient life?
- [ ] Transitions between screens feel smooth (not instant cut)?

---

## 2.6 Cross-Team Juice Spec Template

```markdown
## Juice Spec: [Feature / Event Name]

### Intent
"This moment should feel [adjective — e.g., explosive, warm, satisfying, urgent]"

### Trigger
[Describe what causes this — player input / game event / ambient / transition]

### Effect Sequence
| Step | Delay | Effect | Duration | Easing | Who |
|---|---|---|---|---|---|
| 1 | 0ms | [Effect] | [Xs] | [Curve] | 🛠/🎨/🎲 |

### Motion Specs (if UI)
| Element | Property | From | To | Duration | Easing |
|---|---|---|---|---|---|
| [Name] | Scale | 0 | 1.0 | 0.3s | Elastic |

### Ambient (if applicable)
[Describe any continuous ambient effects]

### Sound
[Description of SFX timing and character]

### Notes for Each Role
- 🛠 Dev: [Code implementation notes]
- 🎨 2D: [UI animation / particle 2D notes]
- 🎲 3D: [VFX / ambient 3D notes]
```
