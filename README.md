# 🎵 Spotify MoodFlow — AI Mood-Based Playlist Engine

> A complete product spec for an AI mood intelligence layer for Spotify — teardown, PRD, competitive analysis, system design, and a fully interactive prototype.

**Author:** Shanit Nagre — AI Product Manager  
**Portfolio:** [shanitnagre.github.io](https://shanitnagre.github.io)

---

## 🔗 Live Prototype

**[Open MoodFlow Interactive Prototype →](https://shanitnagre.github.io/spotify-mood-engine/05-moodflow-prototype.html)**

Full Spotify UI with mood input, energy slider, emotion selection, AI generation animation, now playing screen with real-time progress, and mood-matched queue.

---

## What's Inside

| File | Description |
|------|-------------|
| `01-product-teardown.md` | Spotify's recommendation blind spot — context-free music |
| `02-PRD-moodflow.md` | Full PRD — 6 features, metrics, rollout plan, risks |
| `03-competitive-analysis.md` | Apple Music, YouTube Music, Endel, Brain.fm — who's doing what |
| `04-system-design.md` | AI pipeline: signal ingestion → mood embedding → track selection → adaptation |
| `05-moodflow-prototype.html` | Interactive Spotify UI prototype |

---

## The Core Insight

Spotify's recommendation engine is optimized for **taste continuity** — it gives you more of what you've liked. But music is fundamentally **contextual and emotional**. The same user wants aggressive hip-hop at the gym and ambient instrumentals at 1am. Spotify treats both moments identically.

The result: users default to the same 3 playlists rather than discovering something that fits the moment. A 70% → 13% conversion problem (discovery → actual listening).

MoodFlow closes this gap with an AI layer that captures:
- **Explicit signals** — energy slider + 8 emotion options + destination
- **Behavioral signals** — skip timing, replays, session velocity
- **Contextual signals** — time of day, device, day of week
- **Optional biometrics** — heart rate via Apple Watch / Fitbit

The output: a dynamic playlist that adapts in real-time and converges to your exact micro-mood by track 6–8.

---

## Try the Prototype

1. Open the live link above
2. Tap **"Start MoodFlow"** on the home screen
3. Set your energy level and pick an emotion
4. Choose where you want to go (stay here / lift me up / calm me down)
5. Tap **Generate** — watch the AI generation animation
6. Explore the Now Playing screen — try skipping tracks, using quick presets

---

## Key Design Decisions

**Why 32 dimensions, not 2D valence/arousal?**  
Standard mood models use 2 axes. "Focused" and "Calm" have similar coordinates but need completely different music. 32 dimensions captures social context, activity level, lyric preference, and temporal context separately.

**Why explicit input weights 60%?**  
Passive signals are probabilistic. "11pm" doesn't mean tired — you might be at a party. Explicit input is ground truth.

**Why a 70/30 freshness/familiarity ratio?**  
Too familiar = goes stale. Too new = feels wrong. Research shows 70/30 maximizes engagement, with ratio shifting by mood state.

**Why never deepen negative emotional states by default?**  
Users who feel Spotify made them feel worse churn. Users who feel it helped them through something hard love it forever. Ethical design and good product design align here.

---

*Part of Shanit Nagre's AI PM portfolio — [shanitnagre.github.io](https://shanitnagre.github.io)*
