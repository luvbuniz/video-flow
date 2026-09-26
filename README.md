# Video Flow — Stackadoo Cartoon Pipeline

An automated daily pipeline that produces one sub-60-second educational cartoon
(math or reading) for TikTok, Instagram Reels, and Facebook Reels, to drive
traffic to [Stackadoo](https://stackadoo.com).

Style reference: 2D cartoon characters (not voxel), story-driven mini lessons —
e.g. "the ice cream man shortchanged me, and math saved the day."

---

## Pipeline overview

```
1. IDEA SCOUT ──▶ 2. SCRIPT ──▶ 3. IMAGES ──▶ 4. ANIMATE ──▶ 5. VOICE ──▶ 6. ASSEMBLE ──▶ 7. PACKAGE ──▶ (8. POST)
   trends &         story +       character      image-to-      TTS           captions,       per-platform    future:
   hooks            shot list     keyframes      video clips    narration     music, cut      title/desc/     auto-post
                                                                              to <60s         hashtags
```

Each stage writes its output to a dated folder (`output/2026-07-10/`), so any
stage can be re-run without redoing the whole video.

## Stage details

### 1. Idea scout (daily)
- **What:** Finds what's working in educational short-form content right now and
  proposes 3 ranked concepts (rotating math / reading), each with a hook,
  a relatable story premise, and the lesson payoff.
- **How:** Claude API with the built-in web search tool. It searches trending
  educational content, kids-learning hashtags, and "math trick" / "phonics"
  style videos, then generates original story concepts (we generate ideas
  *inspired by* trends — never copies).
- **Reality check:** TikTok, Instagram, and Facebook do not offer public search
  APIs. Web search + the free YouTube Data API cover trend signals well.
  Optional later: an Apify scraper (~$5–30/mo) for raw TikTok hashtag data.

### 2. Script + shot list
- **What:** Turns the winning concept into a ~55-second script: 8–10 shots,
  each with scene description, character action, dialogue/narration line, and
  on-screen text. Hook in the first 2 seconds; Stackadoo call-to-action at the end.
- **How:** Claude API (`claude-opus-4-8`), structured JSON output so the rest
  of the pipeline can consume it.

### 3. Image generation (keyframes)
- **What:** One keyframe image per shot, plus reusable character reference
  sheets so the kid, the ice cream man, etc. look the same across every episode.
- **How:** GPT Image or Google Imagen / Nano Banana (~$0.02–0.06/image).
  Character consistency comes from feeding the same reference sheet + a fixed
  style prompt into every generation.

### 4. Animation (image-to-video)
- **What:** Each keyframe becomes a 5–8 second animated clip.
- **How (budget):** Kling (~$0.03–0.10/sec) — good motion, cheap.
- **How (premium):** Google Veo 3.1 (~$0.15–0.40/sec) — native audio,
  lip-sync, and dialogue baked in (can replace stage 5 for character lines).

### 5. Voiceover
- **What:** Narration track from the script.
- **How:** ElevenLabs — a consistent, friendly narrator voice. ~1 min/day of
  speech fits their $5/mo Starter plan; the $22/mo Creator plan gives headroom.

### 6. Assembly + captions — [browser-use/video-use](https://github.com/browser-use/video-use)
- **What:** Stitch clips, lay in voiceover + background music, burn in animated
  word-by-word captions (essential for sound-off viewing), export 9:16 1080×1920.
- **Engine: `video-use`** (MIT license, ~16k stars) — an open-source agent
  skill that edits video through Claude Code. FFmpeg renders under the hood;
  the agent plans the cut from word-level transcripts, burns in styled
  subtitles (uppercase 2-word chunks — TikTok style — configurable), adds
  audio fades at every cut, and **self-evaluates the render** (up to 3
  re-render passes) before delivering `edit/final.mp4`.
- **Math bonus:** it can synthesize animation overlays via **Manim** — the
  math-animation library behind 3Blue1Brown — so equations, number lines, and
  counting animations can be layered onto scenes. Also supports Remotion and
  PIL overlays.
- **Transcription:** uses the ElevenLabs Scribe API for word timestamps —
  same ElevenLabs account as our voiceover stage, pennies per video.
- **Fallback / polish:** raw FFmpeg + ASS caption scripts if we ever need a
  fully deterministic path, and optional Kdenlive (free, open source) project
  export for hand-editing a cut.
- Music: royalty-free library (Pixabay/YouTube Audio Library, $0) — or add
  trending platform audio manually at post time.

### 7. Packaging
- **What:** For each platform, generates ready-to-paste posting copy:
  - **TikTok:** hook-style caption, 3–5 hashtags, suggested trending-sound note
  - **Instagram Reels:** caption + hashtag set + cover-frame suggestion
  - **Facebook Reels:** longer parent-oriented caption
  plus a thumbnail/cover image pick and the best posting-time suggestion.
- **How:** Claude API, one call, structured output → `post_kit.md` next to the MP4.

### 8. Auto-posting (future)
Deliberately deferred — for a new account, manual posting is actually better at
first (you can attach trending sounds, reply to comments, and avoid platform
API review delays). When ready:
- **Meta Graph API** — free; posts Instagram Reels + Facebook Pages natively.
- **TikTok Content Posting API** — free but requires app review (~1–2 weeks).
- **Or a scheduler:** Postiz (open source, self-hosted, $0), Buffer
  (~$6/channel/mo), or Blotato (~$29/mo) to hit all three from one place.

---

## Estimated cost — one video per day

Assumes ~55s final video: ~10 keyframes, ~54s of generated animation
(including a 1.5× retry factor — you *will* regenerate some bad clips),
~1 minute of voiceover.

| Stage | Budget tier | Premium tier |
|---|---|---|
| 1. Idea scout + 2. Script + 7. Packaging (Claude API + web search) | $0.30–0.80 | $0.30–0.80 |
| 3. Images (~12 incl. retries) | $0.25–0.60 | $0.50–0.80 |
| 4. Animation (~54s incl. retries) | $2.50–8.00 (Kling) | $12–30 (Veo 3.1) |
| 5. Voiceover (ElevenLabs) | $0.17–0.75 | included in Veo audio |
| 6. Assembly (FFmpeg + Whisper) | $0 | $0 |
| **Per video** | **≈ $3–10** | **≈ $13–32** |
| **Per month (30 videos)** | **≈ $100–300** | **≈ $400–950** |

**Recommendation:** start on the budget tier (~$5/video typical). The animation
model is the only line that really moves the bill — everything else rounds to
pocket change. Upgrade individual "hero" videos to Veo when one concept is
worth extra polish.

### Subscription option — 4 cartoons/month (current plan)

At 4 cartoons/month we need roughly 25–60 video clips a month (6–15 per
cartoon including retries, depending on how much is animated). Recommended
stack (prices as of Sept 2026, monthly billing):

| Need | Pick | Cost/mo |
|---|---|---|
| Images, video, voiceover, music, sound effects, lip sync — **commercial rights + official MCP** | **OpenArt Plus** (12,000 credits) | $34 |
| Caption transcription for video-use | ElevenLabs free account (or faster-whisper locally) | $0 |
| Editing | video-use + FFmpeg | $0 |
| **Total** | | **≈ $34 → ~$8.50 per cartoon** |

**Why Plus, not Starter:** OpenArt's $14 Starter plan does not list commercial
use rights; Plus, Pro, and Wonder do. These videos promote Stackadoo, so
commercial rights are required. Plus's 12,000 credits are far more than
4 cartoons need, leaving room to test models or scale to ~4/week.

Why OpenArt: one credit pool covers images (Nano Banana, GPT Image 2), video
(Seedance 2.0, Kling 3 Omni, Wan, MiniMax), **and audio** — ElevenLabs-powered
text-to-speech (~5 credits per generation), voice clone/changer, music, sound
effects, and lip sync. Seedance 2.0, Kling 3 Omni, and Veo 3.1 can also
generate dialogue + sound effects inside the clip in one pass. Models can be
A/B tested without extra subscriptions, and its MCP server
(`https://mcp.openart.ai/mcp`, OAuth sign-in, no API keys) lets Claude
generate directly.

Cheaper, commercially safe alternative (~$15/month, no MCP, one video model,
more manual work): **Hailuo/MiniMax Standard** ($10 — paid plans include
commercial rights; failed generations still burn credits) + **ElevenLabs
Starter** ($5, commercial license) for the voice. Other options: **Kling**
Standard ($8.80, 660 credits — check its commercial terms) or **Dreamina**
($18 — cheapest direct Seedance, manual web UI only).
Avoid **Higgsfield**: documented complaints about capped "unlimited" plans,
early renewal charges, and a no-refund-once-used policy.

**Credit-saving technique — limited animation:** animate only the 4–6 key
beats of each cartoon; the rest use still keyframes with slow zoom/pan
(free, done at edit time). Roughly halves video credits with little quality loss.
Batch all 4 weekly cartoons in one production session.

## Roadmap

- [ ] v0.1 — Pipeline scripts: idea scout → script → images → clips → FFmpeg cut
- [ ] v0.2 — Caption burn-in (Whisper) + post-kit generation
- [ ] v0.3 — Optional Kdenlive/MLT project export for hand-polish
- [ ] v0.4 — Daily scheduled run (cron) with human approve/reject step
- [ ] v1.0 — Auto-posting via Meta Graph API + TikTok Content Posting API

## Repo layout (planned)

```
video-flow/
├── pipeline/
│   ├── 01_idea_scout.py
│   ├── 02_script.py
│   ├── 03_images.py
│   ├── 04_animate.py
│   ├── 05_voice.py
│   ├── 06_assemble.py
│   └── 07_package.py
├── characters/          # reference sheets + style prompts
├── output/YYYY-MM-DD/   # per-day artifacts (scripts, images, clips, final.mp4, post_kit.md)
└── config.yaml          # model choices, budget tier, brand voice
```
