# PRD: Spotify MoodFlow — AI Mood-Based Playlist Engine
**Author:** Shanit Nagre — AI Product Manager  
**Date:** May 2026  
**Status:** Concept Spec  
**Version:** 1.0

---

## 📌 One-Line Pitch
*An AI engine that reads your current emotional context — through explicit input, listening behavior, time, and optional biometrics — and generates a dynamic playlist that matches where you are and takes you where you want to be.*

---

## 🎯 Problem Statement

### The Core Tension
Spotify's recommendation engine is optimized for taste continuity — it gives you more of what you've liked historically. But music consumption is fundamentally **contextual and emotional**, not just taste-based.

The same user wants:
- Aggressive hip-hop at the gym
- Ambient instrumentals while deep in code
- Nostalgic 2000s pop on a Friday evening
- Nothing with lyrics when anxious

Spotify's engine treats all these moments as the same user with the same needs. The result: users default to the same 3 playlists rather than discovering something that fits the moment.

### The Emotional Intelligence Gap
Music is one of humanity's primary mood regulation tools. People use it deliberately:
- **Mood matching** — play music that reflects how I feel (validation)
- **Mood shifting** — play music that takes me somewhere else (transformation)
- **Mood maintaining** — don't interrupt this state I'm in (continuity)

Spotify has no awareness of which mode the user is in, or what emotional state they're starting from.

---

## 👥 Users

### Primary: Daily Active Streamer (22–35)
- Listens 45+ min/day across multiple contexts
- **Pain:** "I want something that fits right now but I don't know what to search for"
- **Goal:** Music that feels exactly right without effort
- **Success:** Opens Spotify, one tap, perfect music for the moment

### Secondary: Workout/Commute User
- Specific context-driven listening (gym, commute, run)
- **Pain:** Existing workout playlists go stale after a week
- **Goal:** Fresh music that maintains the right energy
- **Success:** Never hits skip during a workout

### Tertiary: Emotional Regulation User
- Uses music deliberately to manage stress, anxiety, or sadness
- **Pain:** Has to manually build playlists for emotional moments
- **Goal:** Music that meets them where they are, then gently shifts
- **Success:** "Spotify knew exactly what I needed"

---

## ✅ Goals & Success Metrics

### North Star
**Session skip rate reduction** — fewer skips = better mood matching = more listening time

### Primary Metrics

| Metric | Baseline | Target (6 months) |
|--------|----------|-------------------|
| Skip rate on MoodFlow sessions | Avg 28% | ≤ 14% |
| Session length (MoodFlow vs standard) | 28 min | ≥ 38 min |
| MoodFlow DAU / total DAU | — | ≥ 25% |
| 30-day retention (MoodFlow users) | — | +18% vs control |

### Secondary Metrics
| Metric | Target |
|--------|--------|
| Mood input completion rate | ≥ 70% |
| New artist discovery rate (MoodFlow) | +40% vs standard |
| Premium conversion (free users via MoodFlow) | +8% |

### Guardrail Metrics
- Overall skip rate: must not increase
- Session start rate: must not decrease
- Explicit mood input abandonment: < 30%

---

## 🔧 Feature Breakdown

### Feature 1: MoodFlow Entry Point
**What:** A new "How are you feeling?" entry point on the home screen and now-playing screen.

**UX:**
- Persistent but subtle — a floating pill at the bottom of home screen
- Tap → 3-step mood capture (10 seconds max):
  1. **Energy level** — slider from 0 (exhausted) to 10 (electric)
  2. **Emotional state** — 8 emoji options (😤 stressed, 😢 sad, 😊 happy, 😌 calm, 💪 motivated, 🥱 tired, 🥳 celebratory, 😶 numb)
  3. **Mood destination** (optional) — "Stay here" or "Take me to [state]"

**Output:** MoodFlow generates a 2-hour dynamic playlist in < 3 seconds

---

### Feature 2: Passive Mood Detection (Background)
**What:** AI infers likely mood from behavioral signals without requiring explicit input.

**Signals used:**
| Signal | What it indicates |
|--------|------------------|
| Time of day | Morning = energizing, late night = wind-down |
| Day of week | Monday morning vs Friday evening |
| Session start pattern | Rapid skipping = unsettled, steady listening = calm |
| Recent skip history | What was rejected recently |
| Device type | Phone (mobile) vs speaker (social) vs earphones (personal) |
| Listening velocity | Fast-forward heavy = impatient/energized |
| Recent weather (opt-in location) | Rainy day = introspective signal |

**Output:** Suggested mood state shown subtly — "Sounds like you need something energizing. Try MoodFlow →"

---

### Feature 3: Dynamic Playlist Adaptation
**What:** Playlist adjusts in real-time based on engagement signals during the session.

**How it works:**
- Skip at < 10s → song dramatically wrong, avoid this region of mood space
- Skip at 1–2 min → direction right, energy level off
- Full listen + immediate replay → perfect match, anchor here
- Full listen, no replay → good but move forward
- Session pause > 5 min → re-evaluate mood on return

**Result:** By track 5–6, MoodFlow has calibrated to the exact micro-mood. By track 10, it's essentially a real-time mood mirror.

---

### Feature 4: Mood Journey Mode
**What:** For users who want to deliberately shift emotional state over time.

**Use cases:**
- "I'm anxious before a presentation — take me to calm and focused over 30 minutes"
- "I just broke up with someone — let me feel it, then gradually lift me"
- "Post-workout — bring me down slowly from high energy"

**UX:**
- Start state + end state + duration
- Playlist arc: starts matching current mood, transitions gradually toward destination
- BPM, key, energy level, and lyrical content all shift progressively
- User can see the "mood arc" visualized as a curve

---

### Feature 5: Context Presets
**What:** One-tap context modes that set mood parameters automatically.

| Preset | Energy | Mood | BPM Range | Lyric preference |
|--------|--------|------|-----------|-----------------|
| 🏃 Running | 9/10 | Motivated | 150–175 | Hype |
| 📚 Focus | 4/10 | Calm | 60–90 | Instrumental preferred |
| 😴 Sleep | 1/10 | Calm | 40–65 | Minimal/none |
| 🍽️ Dinner | 5/10 | Happy | 90–110 | Tasteful |
| 💪 Gym | 8/10 | Aggressive | 130–160 | Hype |
| 🚗 Commute | 6/10 | Neutral | 100–130 | Familiar |
| 🥳 Party | 9/10 | Celebratory | 120–140 | Party anthems |
| 😢 Processing | 3/10 | Sad | 60–80 | Emotional |

---

### Feature 6: Biometric Integration (Phase 2, opt-in)
**What:** Use Apple Watch / Fitbit heart rate data to infer emotional arousal level.

**Signals:**
- Resting HR + current HR = stress/excitement level
- HRV = emotional regulation state
- Movement detection = activity context

**Privacy:** Entirely opt-in, processed on-device where possible, user can delete at any time.

---

## 📐 Technical Requirements

### Mood-to-Music Mapping Model
```
INPUT SIGNALS
├── Explicit: energy (0-10), emotion (8 categories), destination
├── Behavioral: time, day, device, skip patterns, session velocity  
├── Historical: user's mood-listening correlations (personalized)
└── Optional: biometrics, location/weather

        ↓

MOOD EMBEDDING
├── Map signals to 32-dimensional mood vector
├── Personal calibration (your "happy" ≠ everyone's "happy")
└── Confidence score per dimension

        ↓

TRACK SELECTION ENGINE  
├── Audio feature matching (valence, energy, danceability, tempo, acousticness)
├── Lyrical sentiment matching (NLP on lyrics)
├── Freshness vs familiarity balance (70/30 default, user-adjustable)
└── Transition smoothness (adjacent tracks must be mood-compatible)

        ↓

DYNAMIC ADAPTATION LOOP
├── Real-time skip signal processing
├── Playlist re-ranking every 3 tracks
└── Mood drift detection (is the user's mood shifting?)
```

### Spotify Audio Features Used
Spotify's existing audio analysis API already provides:
- **Valence** (0–1): Musical positiveness
- **Energy** (0–1): Perceptual intensity
- **Danceability** (0–1): Rhythmic suitability for dancing
- **Tempo** (BPM): Beats per minute
- **Acousticness** (0–1): Acoustic vs electronic
- **Instrumentalness** (0–1): Predicts no vocals
- **Loudness** (dB): Overall loudness

**The gap:** These features exist but aren't being used to map to emotional states in real-time. MoodFlow closes this gap.

### Performance Requirements
| Requirement | Target |
|-------------|--------|
| Playlist generation latency | < 3 seconds |
| Real-time adaptation latency | < 500ms per skip event |
| Mood vector computation | < 200ms |
| Playlist length | 2 hours (refreshed if needed) |

---

## 🗺️ Phased Rollout

### Phase 1 — Explicit Mood Input (Month 1–3)
- Manual mood input only (energy + emotion sliders)
- 10% of Premium users in US + UK + India
- **Go/No-Go:** Skip rate reduction ≥ 8%, MoodFlow session length ≥ 35 min

### Phase 2 — Passive Detection + Dynamic Adaptation (Month 4–6)
- Add passive signal detection
- Add real-time playlist adaptation
- **Go/No-Go:** MoodFlow DAU ≥ 20% of total DAU

### Phase 3 — Mood Journey + Context Presets (Month 7–9)
- Full mood journey mode
- Context presets launched
- **Go/No-Go:** 30-day retention lift ≥ 15%

### Phase 4 — Biometric Integration (Month 10–12)
- Apple Watch / Fitbit opt-in
- **Go/No-Go:** Biometric user skip rate ≤ 10%

---

## ⚠️ Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Users find mood input friction too high | High | High | Make it 2 taps max, skip option always available |
| Model gets mood wrong → user frustration | High | High | Easy "Not quite" feedback → instant re-generation |
| Privacy concerns around emotional data | Medium | High | Clear consent, local processing where possible, no selling to advertisers |
| Mood labeling feels prescriptive/clinical | Medium | Medium | Use emoji + energy rather than clinical mood terms |
| Biometric data security breach | Low | Critical | On-device processing, no raw biometric storage |

---

*Shanit Nagre · AI Product Manager · [shanitnagre.github.io](https://shanitnagre.github.io)*
