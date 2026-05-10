# System Design: Spotify MoodFlow — AI Mood Engine
**Author:** Shanit Nagre — AI Product Manager  
**Date:** May 2026

---

## Overview

MoodFlow is a real-time mood intelligence layer that sits between Spotify's existing audio catalog and the user's listening session. It ingests multi-modal signals, generates a mood embedding, maps it to audio feature space, and continuously adapts the playlist based on engagement feedback.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     SIGNAL INGESTION LAYER                      │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │ Explicit     │  │ Behavioral   │  │ Contextual             │ │
│  │ Mood Input   │  │ Signals      │  │ Signals                │ │
│  │              │  │              │  │                        │ │
│  │ • Energy 0-10│  │ • Skip rate  │  │ • Time of day          │ │
│  │ • Emotion    │  │ • Skip timing│  │ • Day of week          │ │
│  │ • Destination│  │ • Replays    │  │ • Device type          │ │
│  │              │  │ • Session    │  │ • Weather (opt-in)     │ │
│  │              │  │   velocity   │  │ • Recent history       │ │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬─────────────┘ │
│         └─────────────────┴──────────────────────┘              │
│                            │                                     │
│                   ┌────────▼────────┐                           │
│                   │ Optional:        │                           │
│                   │ Biometric Input  │                           │
│                   │ (HR, HRV, steps) │                           │
│                   └────────┬────────┘                           │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MOOD EMBEDDING ENGINE                         │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Signal Fusion Model (Transformer-based)                  │   │
│  │  • Weights explicit > behavioral > contextual             │   │
│  │  • Personal calibration layer (your baseline)             │   │
│  │  • Confidence score per dimension                         │   │
│  └──────────────────────────┬───────────────────────────────┘   │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────────────┐   │
│  │  32-Dimensional Mood Vector                               │   │
│  │  [energy, valence, arousal, stress, focus,                │   │
│  │   social_context, activity, temporal_context ...]         │   │
│  └──────────────────────────┬───────────────────────────────┘   │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  TRACK SELECTION ENGINE                          │
│                                                                  │
│  ┌──────────────────┐    ┌─────────────────────────────────┐    │
│  │ Spotify Audio    │    │ Mood-to-Audio Feature Mapper     │    │
│  │ Features Index   │◀───│                                  │    │
│  │                  │    │ Mood Vector → Audio Feature      │    │
│  │ 100M+ tracks     │    │ Target Range                     │    │
│  │ • Valence        │    │                                  │    │
│  │ • Energy         │    │ e.g. stressed + wants calm →     │    │
│  │ • Danceability   │    │ valence: 0.6-0.8                 │    │
│  │ • Tempo          │    │ energy: 0.2-0.4                  │    │
│  │ • Acousticness   │    │ tempo: 60-80 BPM                 │    │
│  │ • Instrumentalness│   │ instrumentalness: 0.6+           │    │
│  └──────────────────┘    └─────────────────────────────────┘    │
│              │                                                   │
│              ▼                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Candidate Track Retrieval (ANN Search)                   │   │
│  │  → Filter by audio feature ranges                         │   │
│  │  → Filter by user's taste profile (avoid hated artists)   │   │
│  │  → Filter by freshness target (70% new, 30% familiar)     │   │
│  │  → Apply lyrical sentiment filter (NLP)                   │   │
│  └──────────────────────────┬───────────────────────────────┘   │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────────────┐   │
│  │  Transition Smoothness Scorer                             │   │
│  │  → Adjacent tracks must be mood-compatible                │   │
│  │  → No jarring energy jumps (max 0.3 delta per track)      │   │
│  │  → Key compatibility check (avoid clashing keys)          │   │
│  └──────────────────────────┬───────────────────────────────┘   │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  REAL-TIME ADAPTATION LOOP                       │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │  Engagement Signal Processor                            │     │
│  │                                                         │     │
│  │  Skip < 10s    → "Wrong direction" signal (strong)      │     │
│  │  Skip 10s-1min → "Energy level off" signal (medium)     │     │
│  │  Skip 1-2min   → "Right vibe, minor adjustment"         │     │
│  │  Full listen   → "Good match" signal                    │     │
│  │  Replay        → "Perfect match" anchor signal          │     │
│  │  Pause > 5min  → Re-evaluate mood on return             │     │
│  └────────────────────────┬───────────────────────────────┘     │
│                           │                                      │
│  ┌────────────────────────▼───────────────────────────────┐     │
│  │  Playlist Re-Ranking Engine                             │     │
│  │  • Re-ranks queue every 3 tracks                        │     │
│  │  • Mood vector updated with each skip signal            │     │
│  │  • Converges to user's exact micro-mood by track 6-8    │     │
│  └────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Design Decisions

### Decision 1: 32-Dimensional Mood Vector (not 2D valence/arousal)

**Standard approach:** Russell's Circumplex Model maps all emotions to 2 dimensions: valence (positive/negative) and arousal (high/low energy). Simple, well-studied.

**Problem:** Too coarse. "Focused" and "Calm" have similar valence and arousal but need very different music. "Angry" and "Excited" are both high arousal but are completely different listening states.

**MoodFlow approach:** 32 dimensions including: social context (alone vs social), activity (stationary vs moving), temporal context (morning vs night), stress level (separate from energy), focus requirement, and domain-specific dimensions (nostalgia appetite, lyric preference).

**Trade-off:** More complex model, harder to explain to users. Mitigated by: users never see the vector — they see emoji and energy sliders.

---

### Decision 2: Explicit Input Weights Higher Than Passive Signals

**Why:** Passive signals (time of day, skip rate) are probabilistic. "It's 11pm" doesn't mean you're tired — you might be at a party. Explicit input ("I'm feeling energetic") is ground truth.

**Weighting:**
- Explicit input: 60% weight
- Recent behavioral signals: 25% weight
- Contextual signals: 15% weight
- Biometric (when available): overrides contextual, adds to behavioral

**Fallback:** If no explicit input, fall back to passive + contextual signals with a "Does this feel right?" prompt after track 3.

---

### Decision 3: 70/30 Freshness/Familiarity Balance

**The tension:** Too much new music = feels like a stranger's playlist. Too much familiar music = no discovery value, goes stale.

**Research basis:** Spotify's own research shows optimal engagement when ~30% of session tracks are familiar ("comfort anchors") and ~70% are new. This ratio shifts by mood state:
- High stress / emotional processing → 50/50 (need anchors)
- High energy / celebratory → 80/20 (want newness)
- Focus mode → 60/40 (familiar instrumentals preferred)

**Implementation:** User-adjustable slider in settings: "Familiar ←→ Adventurous"

---

### Decision 4: No "Sad" Trap — Handle Emotional Vulnerability Carefully

**The problem:** Music for emotional processing (grief, breakup, sadness) is one of the most common mood-based listening use cases. But it can also reinforce negative emotional states if handled poorly.

**Design principle:** MoodFlow never deepens negative emotional states without user intent.
- Default for "sad" input: gentle matching, then gradual lift over 45 minutes
- Explicit "Stay in this feeling" option available but not default
- After 60 minutes of low-valence listening: subtle check-in prompt ("Still feeling this? Or ready to shift?")

**Why this matters:** This is both ethical design and good product design. Users who feel Spotify made them feel worse will churn. Users who feel Spotify helped them through something hard will love it forever.

---

### Decision 5: On-Device Processing for Biometric Data

**Problem:** Heart rate and HRV are sensitive health data. Sending to servers creates privacy risk and regulatory complexity (HIPAA-adjacent concerns).

**Decision:** Biometric signal processing happens on-device (iPhone/Android). Only the derived mood signal (a number between 0–10) is sent to Spotify's servers. Raw biometric data never leaves the device.

**Trade-off:** Limits the sophistication of biometric models (can't run large models on-device). Acceptable — we need the arousal estimate, not clinical precision.

---

## Failure Modes & Handling

| Failure | Detection | Response |
|---------|-----------|----------|
| Wrong mood detected | 3+ skips in first 5 tracks | "Not quite right? Tell us how you're feeling" prompt |
| Mood model cold start (new user) | < 10 listening history entries | Fall back to explicit input only, no passive inference |
| Biometric data unavailable | Device not paired | Graceful degradation to behavioral + contextual signals |
| No matching tracks found | < 5 candidates in audio feature range | Relax constraints gradually until minimum 20 tracks found |
| User mood shifts mid-session | Sudden skip pattern change after stable session | Re-trigger mood assessment prompt after 2 consecutive skips |

---

## PM Health Metrics

| Metric | Alert Threshold | Action |
|--------|----------------|--------|
| Skip rate on MoodFlow sessions | > 20% | Review mood-to-audio mapping model |
| Mood input abandonment | > 40% | Simplify input UI |
| Session length (MoodFlow) | < 30 min avg | Review playlist quality |
| "Not quite right" tap rate | > 25% | Passive signal weights need recalibration |
| Negative mood deepening events | > 5% of sad sessions | Review emotional safety guardrails |

---

*Shanit Nagre · AI Product Manager · [shanitnagre.github.io](https://shanitnagre.github.io)*
