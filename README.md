<div align="center">

# LINGUA//GRID

**A TRON-styled, browser-based multilingual pronunciation runner**

*Ride a neon light-cycle down a grid highway. Ten barriers rise, each showing a phrase in a different language. Speak it acceptably to drop the barrier. Lowest final time wins.*

Built for public and student engagement events (Republic Polytechnic, Singapore).
Vanilla HTML/CSS/JS · No build step · No install · Runs on GitHub Pages.

</div>

---

## Table of contents

1. [What it is](#1-what-it-is)
2. [Quick start](#2-quick-start)
3. [How to play](#3-how-to-play)
4. [Technical architecture](#4-technical-architecture)
5. [The scoring engine](#5-the-scoring-engine)
6. [Languages, difficulty and selection](#6-languages-difficulty-and-selection)
7. [Data, privacy and the shared leaderboard](#7-data-privacy-and-the-shared-leaderboard)
8. [Configuration reference](#8-configuration-reference)
9. [Deployment](#9-deployment)
10. [Testing](#10-testing)
11. [Scientific and clinical rationale](#11-scientific-and-clinical-rationale)
12. [Educational and cultural value](#12-educational-and-cultural-value)
13. [Limitations and honest caveats](#13-limitations-and-honest-caveats)
14. [References](#14-references)
15. [Accessibility, credits and licence](#15-accessibility-credits-and-licence)

---

## 1. What it is

LINGUA//GRID is a lightweight, single-file web game that invites players to **pronounce short phrases across many world languages**. It is designed to be *instant, loud, forgiving and fun* — a "TikTok-style" mini-game feel — while quietly encouraging players to hear, attempt and appreciate the sounds of languages they may never have spoken before.

The game uses the browser's own speech-recognition service to estimate how *intelligible* each attempt was, awards a "SYNC %" score, and drops a neon barrier when the attempt is good enough. It is deliberately lenient: the goal is engagement and exposure, not examination.

**Design pillars**

1. **Fun first** — lenient pass thresholds, big feedback, combo and "perfect" call-outs.
2. **Never stuck** — unlimited retries, an audio hint (time penalty), and a skip after repeated failure.
3. **Honest data** — every attempt's transcript and similarity score is recorded.
4. **Zero setup** — open a URL, allow the microphone, play.

---

## 2. Quick start

> The microphone requires a **secure context**. Serve the page over **HTTPS** (e.g. GitHub Pages) or **localhost**. Opening the file directly with `file://` will load the visuals but the browser will block microphone access.

**Play the hosted version**
Open the deployed URL in **desktop Google Chrome or Microsoft Edge** (the two supported browsers), allow the microphone when prompted, and play.

**Run locally**
Because the mic needs a secure context, serve the folder over localhost with any static server, for example:

```bash
# any static file server works; pick one you already have
npx serve .            # Node
python -m http.server  # Python 3
```

Then open `http://localhost:<port>/index.html`.

> **Recommended for events:** use **Microsoft Edge**. Its online neural voices cover far more of the supported languages than Chrome on Windows, which improves both hint playback and language availability (see §6).

---

## 3. How to play

1. **Preflight** — the game checks for speech-recognition support, a secure context, available voices, and builds the language pool.
2. **Register** — enter name, age and gender, pick a **difficulty**, and give consent (PDPA notice).
3. **Mic check** — allow the microphone. Once the level meter responds to your voice you can enter the grid; saying "ready" is an optional confirmation, not a gate.
4. **Ride** — ten barriers rise in turn. Each shows a country flag, the language name, the phrase in its native script, a romanisation (for non-Latin scripts) and an IPA transcription.
5. **Speak** — press **SPACE** (or the SPEAK button) and say the phrase. A live waveform and interim transcript appear; the SYNC ring fills with your score.
6. **Pass, hint or skip**
   - **Pass** — score ≥ the language threshold drops the barrier; the cycle boosts.
   - **Hint (H)** — plays the phrase audio; each use adds a time penalty.
   - **Skip** — offered after repeated failures; adds a larger time penalty and records the stage as skipped.
7. **Finish** — the timer runs from the first barrier to the last. Final time = elapsed + penalties − perfect bonuses. Results show a per-stage breakdown, your rank, and the leaderboard.

**Keyboard shortcuts:** `SPACE` speak · `H` hint · `S` skip (when offered) · `Ctrl+Shift+L` admin/data panel.

---

## 4. Technical architecture

LINGUA//GRID is a **single, self-contained `index.html`** (inline CSS and JavaScript) plus a `tests.html` test page. There is no build step, no package manager, and no framework.

### Hard constraints (by design)
- Vanilla HTML/CSS/JS in one file; the author cannot install software locally.
- Hosted on GitHub Pages (HTTPS, required for microphone access).
- Target browsers: desktop Chrome and Edge (latest). Any other browser shows an "unsupported" screen.
- Only two runtime externals, both from a CDN and **version-pinned**:
  - **Google Fonts** — Orbitron (UI) and the Noto Sans families (native scripts and IPA).
  - **`pinyin-pro@3.26.0`** — tone-insensitive Mandarin matching.
- **No emoji flags** (they render as letters on Windows) — every flag is a hand-authored inline SVG.
- No image or audio assets are required to run. Optional pre-generated hint MP3s can live in `/audio/{phraseId}.mp3`.

### Browser APIs used
| Capability | API |
|---|---|
| Speech-to-text | `webkitSpeechRecognition` / `SpeechRecognition` (cloud-backed in Chrome/Edge) |
| Microphone level / waveform | `navigator.mediaDevices.getUserMedia` + Web Audio `AnalyserNode` |
| Hint playback fallback | `speechSynthesis` |
| Highway renderer | Canvas 2D (pseudo-3D, glow via `shadowBlur` + additive compositing) |
| Sound effects | Web Audio oscillators (fully synthesised, no files) |
| Storage | `localStorage`; optional `fetch` to a leaderboard webhook |

### Module layout
The script is organised as plain objects / IIFEs in one file:

```
Game (state machine)
 ├─ Config / Phrases / Flags / Langs   data + hand-drawn SVG flags
 ├─ Scorer      pure functions: normalize, graphemes, levenshtein, similarity
 ├─ Speech      SpeechRecognition wrapper → Promise<Result>; mic analyser
 ├─ Audio       hint playback (MP3 → speechSynthesis fallback) + synth SFX
 ├─ Renderer    Canvas 2D highway, light-cycle, barriers, particles (60 fps target)
 ├─ Store       localStorage runs, CSV/JSON export, webhook POST + retry queue
 ├─ UI          DOM overlays: screens, HUD, barrier card, callouts, toasts
 ├─ Eligibility language eligibility + weighted pool building
 └─ Results     results screen + shared/local leaderboard
```

### State machine
```
BOOT → UNSUPPORTED | REGISTER → MIC_CHECK → COUNTDOWN → RIDING → BARRIER_RISE
     → AWAIT_SPEAK ⇄ LISTENING → (PASS → RIDING | FAIL → AWAIT_SPEAK) …
     → FINISH → RESULTS → LEADERBOARD
```
The run timer starts at the first barrier and stops when the last barrier is passed, including hint playback and listening time.

### Rendering
A Canvas 2D "pseudo-3D" highway: a horizon at ~40% height, a road trapezoid with speed-driven scrolling lines and converging lanes, a parallax wireframe skyline, a vector light-cycle with a rear light-trail ribbon, rising orange barriers with a scan-line animation, and particle shatter on a pass. Glow is achieved with `shadowBlur` and `globalCompositeOperation = 'lighter'`. The loop is `requestAnimationFrame`-driven with delta-time scaling and honours `prefers-reduced-motion`.

---

## 5. The scoring engine

The `Scorer` is a set of **pure, unit-tested functions**. It measures **intelligibility** — did the recogniser understand you — which is the same pragmatic approach casual pronunciation games use. It is explicitly *not* a phoneme-level clinical pronunciation assessment.

**Pipeline**

1. **Normalize** (per language):
   - Unicode NFC, lower-case.
   - Arabic: strip tashkeel/tatweel; Hebrew: strip niqqud/cantillation.
   - Latin-script languages (except Vietnamese, whose tone diacritics are meaningful): strip diacritics.
   - Remove punctuation and symbols; collapse whitespace.
   - Space-free scripts (Chinese, Thai, Burmese, Japanese): remove whitespace.
   - Mandarin: convert to **toneless pinyin** via `pinyin-pro`, so homophones and tone variation do not unfairly fail a learner.
2. **Grapheme segmentation** with `Intl.Segmenter` (correctly handles combining marks and complex scripts).
3. **Similarity** = `1 − levenshtein(graphemes(a), graphemes(b)) / max(len)`.
4. **Window similarity** — tolerates extra filler words ("um, bonjour, comment ça va") by scanning sub-spans of the transcript.
5. **Score** = the best similarity across up to five recognition alternatives and all accepted phrase variants.

A stage passes when the best similarity ≥ the language threshold (default **0.65**; lowered to **0.55** for Thai, Burmese and Tamil, whose recognisers are less reliable). A score ≥ **0.90** triggers a "PERFECT" call-out and a time bonus. All thresholds live in `CONFIG`.

---

## 6. Languages, difficulty and selection

**Supported languages (17 in the main pool):** English, Mandarin, Tamil, Hindi, Bahasa Melayu, Bahasa Indonesia, Thai, Burmese, Vietnamese, French, German, Spanish, Hebrew, Arabic, Russian, **Japanese, Korean**. Additional backup languages (Italian, Portuguese, Filipino) fill in if too few of the pool are available.

Stage 1 is always **English**; stages 2–10 are chosen from the remaining eligible languages.

**Eligibility.** A language can only appear if the player's browser can actually voice it — that is, a matching `speechSynthesis` voice exists, or a bundled hint MP3 is present. This prevents a barrier the player cannot get a hint for. On boot, the console logs the full eligible list and explicitly reports whether Japanese and Korean are available, so event organisers can confirm coverage (Edge covers far more languages than Chrome on Windows).

**Weighted selection.** Selection is weighted random *without replacement*. Weights are configurable; by default Japanese and Korean are up-weighted and Arabic is down-weighted (it is phonetically challenging for most newcomers), to tune the difficulty and cultural mix of a typical run.

**Difficulty levels** (chosen at registration):
- **EASY** — single words.
- **MEDIUM** — short everyday phrases.
- **INTERMEDIATE** — complete everyday sentences.

Each phrase is tagged with a level; if a language lacks a phrase at the chosen level, the game falls back gracefully so no stage is ever unplayable. Leaderboards are ranked **per difficulty** so runs are compared fairly.

> **Content note.** IPA and romanisation for the seed phrase bank are broad transcriptions. New entries added for the EASY/INTERMEDIATE tiers are flagged for **native-speaker verification** before competitive or public use.

---

## 7. Data, privacy and the shared leaderboard

**What is stored.** For each completed run: player name, age and gender; final time, penalties and bonuses; and a per-stage log (language, phrase, recognised transcript, similarity, attempts, pass/skip). Data is stored in the browser's `localStorage`.

**What is *not* stored.** **No audio recordings are ever kept.** Speech is sent to the browser's cloud speech service purely to produce a text transcript, which is scored locally.

**Consent (Singapore PDPA).** Registration presents a data-use notice and requires an explicit consent checkbox before play. Player names are stripped of HTML before storage or display.

**Shared leaderboard (optional).** When a leaderboard webhook (e.g. a Google Apps Script bound to a Google Sheet) is configured in `CONFIG`, each completed run is `POST`ed to it, and the leaderboard screen `GET`s the combined top-10 across all devices. The on-screen board **merges** the shared results with the local ones and falls back to local-only when offline, so a player always sees their own just-finished run even if the sheet has not yet synced. A queue retries any failed sends on the next load.

**Admin panel** (`Ctrl+Shift+L`): export all runs as CSV (one row per stage) or JSON, retry the webhook queue, or clear stored data after confirmation.

---

## 8. Configuration reference

Every tunable lives in a single `CONFIG` object at the top of the script:

| Key | Purpose |
|---|---|
| `stages` | Number of barriers per run (default 10). |
| `firstLanguage` | Always the first stage (default `en`). |
| `pool` / `backupLanguages` | Main and fallback language sets. |
| `weights` | Relative selection probability per language. |
| `defaultLevel` / `levels` / `levelFallback` | Difficulty definitions and fallback order. |
| `threshold` | Pass threshold overall and per language. |
| `perfectAt` / `perfectBonusMs` | "Perfect" cut-off and time bonus. |
| `hintPenaltyMs` / `skipPenaltyMs` / `skipAfterFails` | Penalties and skip gating. |
| `maxListenMs` | Maximum listening window per attempt. |
| `audioPath` / `useBundledAudio` | Optional bundled hint MP3s. |
| `leaderboardWebhook` | URL for the shared leaderboard (empty = local only). |
| `storageKey` | `localStorage` namespace. |

---

## 9. Deployment

**GitHub Pages (recommended).**
1. Create a public repository (e.g. `lingua-grid`).
2. Upload `index.html` and `tests.html` to the repository root (and an `audio/` folder later if you generate hint MP3s).
3. **Settings → Pages → Deploy from a branch**, select `main` / `/ (root)`, Save.
4. Open `https://<user>.github.io/<repo>/` in Chrome or Edge (HTTPS enables the microphone).

Only `index.html` is required to run; `tests.html` is optional but recommended. The initial load is well under 2 MB excluding optional audio.

---

## 10. Testing

Open **`tests.html`** in the browser. It loads the same pinned `pinyin-pro` CDN and a verbatim copy of the `Scorer`, then asserts a suite of cases including:

- Exact matches score 1.0; punctuation and case are ignored.
- Arabic with/without tashkeel and Hebrew with/without niqqud score ≈ 1.0.
- Mandarin homophones match via toneless pinyin.
- Filler words around a target still pass; a wrong phrase scores low.
- Threshold table, difficulty-level selection and fallbacks, weighted-selection ordering, and the mic-check acceptance logic.

A green **"ALL n TESTS PASSED"** banner indicates success.

---

## 11. Scientific and clinical rationale

> **Framing.** LINGUA//GRID is an **engagement and outreach tool**, not a medical device or a clinical intervention. The evidence below explains *why exposure to and playful practice with multiple languages is a worthwhile activity*, and it is presented with the genuine scientific caveats intact. We deliberately avoid overclaiming.

### 11.1 Language experience and cognitive reserve

The strongest clinical signal in this area concerns **cognitive reserve** — the brain's capacity to tolerate age- or disease-related change before symptoms appear. In a large clinical cohort in Hyderabad, India, Alladi and colleagues reported that **bilingual patients developed dementia on average about 4.5 years later than monolingual patients**, an effect that held across dementia subtypes and was independent of education, sex, occupation and urban/rural residence, and was even observed in illiterate patients — arguing against a purely socioeconomic explanation (Alladi et al., 2013, *Neurology*). Reviews by Bialystok and colleagues frame lifelong bilingualism as a contributor to cognitive reserve that can **delay the onset of dementia symptoms by several years** (Bialystok, Craik & Luk, 2012, *Trends in Cognitive Sciences*; Bialystok, 2021).

This matters for public health. The **2020 Lancet Commission on dementia prevention** estimated that around **40% of dementia cases are potentially preventable or delayable** by addressing 12 modifiable risk factors across the life course — the first of which is **low education / cognitive engagement** (Livingston et al., 2020, *The Lancet*). Cognitively stimulating, socially engaging activities are a recognised part of that picture. A multilingual pronunciation game is a small, enjoyable instance of exactly this kind of novel cognitive and social stimulation.

### 11.2 Speech perception, phonetic training and neuroplasticity

Learning to *hear* and *produce* the sounds of an unfamiliar language is a genuine perceptual-motor challenge. Adults can and do improve at discriminating non-native speech contrasts with training, and such **high-variability phonetic training** produces measurable, retained gains — the classic demonstration being native Japanese speakers learning the English /r/–/l/ contrast (Logan, Lively & Pisoni, 1991, *Journal of the Acoustical Society of America*; Lively, Logan & Pisoni, 1993). Short, repeated, feedback-rich exposure — precisely the loop this game provides — is the format that phonetic-training research uses.

At the neural level, acquiring and using more than one language is associated with **structural and functional brain differences**, including changes in grey-matter density and white-matter integrity in regions supporting language and executive control (Mechelli et al., 2004, *Nature*; García-Pentón et al., 2014). Musical and linguistic auditory training more broadly are linked to enhanced auditory and speech-in-noise processing (Kraus & Chandrasekaran, 2010, *Nature Reviews Neuroscience*), underscoring that structured practice with sound is a real form of cognitive exercise.

### 11.3 Executive function — a balanced view

It is often claimed that bilingualism confers a general "executive-function advantage." **The honest scientific position is that this is contested.** Early studies reported bilingual advantages in inhibition and task-switching, but a series of large analyses found the effects to be **small, inconsistent, task- and age-dependent, and inflated by publication bias** (Paap & Greenberg, 2013, *Cognitive Psychology*; de Bruin, Treccani & Della Sala, 2015, *Psychological Science*; Lehtonen et al., 2018, *Psychological Bulletin*). Where effects do appear, they are most detectable in **older adults**, consistent with the cognitive-reserve interpretation rather than a large everyday IQ-style boost.

**We therefore make no claim that playing this game will make anyone measurably "smarter."** What the literature does support is more modest and still worthwhile: multilingual experience is a form of enriching cognitive and social activity, and lifelong language engagement is associated with delayed clinical dementia onset via cognitive reserve.

### 11.4 Why gamification and why this design

Games are effective engagement vehicles because they align with well-established motivational and learning principles: immediate feedback, achievable challenge, and low stakes for error. LINGUA//GRID applies these deliberately — lenient thresholds, unlimited retries, "perfect"/combo call-outs, and a hint that keeps players moving — which lowers the anxiety that typically accompanies speaking an unfamiliar language, itself a known barrier to second-language production (Horwitz, Horwitz & Cope, 1986, *The Modern Language Journal*, on foreign-language anxiety). The result is a setting in which a member of the public will happily *attempt* Tamil, Burmese or Korean out loud — the crucial first step toward interest, exposure and, potentially, further learning.

---

## 12. Educational and cultural value

Beyond cognition, LINGUA//GRID is built to foster **linguistic curiosity and cross-cultural appreciation** — especially apt in a multilingual society such as Singapore, whose four official languages and many community tongues are a daily reality.

- **Exposure breadth.** In a few minutes, a player hears and attempts up to ten languages spanning several scripts and phonological systems — from tonal Mandarin, Thai and Vietnamese to the retroflexes of Tamil, the abugidas of Hindi and Burmese, and the RTL scripts of Hebrew and Arabic.
- **Respectful presentation.** Each barrier pairs the native script with a romanisation and IPA, and pronounces the phrase on demand, so players engage with a language *as its speakers write and say it*, not merely a transliteration.
- **Lowering the "I could never say that" barrier.** The forgiving scoring and playful framing convert intimidation into a game, which is exactly the mindset shift that precedes genuine interest in a language and its culture.
- **For aspiring polyglots.** The difficulty tiers (single words → phrases → full sentences), the per-language pronunciation feedback, and the honest per-attempt data make the game a low-friction *warm-up and self-monitoring* tool: it will not teach grammar, but it builds the ear, the confidence and the habit of attempting many languages — the foundation of a polyglot's practice.
- **Community and events.** The shared leaderboard turns a solo activity into a friendly, cross-cultural competition, encouraging conversation about the languages themselves among participants.

In short, the game's educational contribution is **motivational and affective** — it makes the sounds of the world approachable and enjoyable — which is a legitimate and evidence-aligned goal for a public-engagement tool.

---

## 13. Limitations and honest caveats

- **Not an assessment.** The SYNC score measures whether a cloud recogniser *understood* an utterance, not phonetic accuracy or fluency. High scores are encouraging, not certification.
- **Recogniser and voice coverage vary by browser and OS.** Some languages may be unavailable (no voice), and recognition quality differs markedly between languages. Edge generally offers the widest coverage on Windows.
- **Internet required.** Speech recognition is cloud-backed; the fonts and `pinyin-pro` load from a CDN.
- **Cognitive-health evidence is about sustained, lifelong multilingual experience**, not about any single game session. Claims here are framed accordingly; the game is an on-ramp to engagement, not a treatment.
- **The bilingual executive-function "advantage" is scientifically contested** and is not relied upon in this document.
- **Seed content needs verification.** Newly added IPA/romanisation should be checked by native speakers before high-stakes use.

---

## 14. References

*Citations are to peer-reviewed literature; readers are encouraged to consult the originals. Content in this section was summarised and paraphrased from the sources for compliance with licensing restrictions.*

1. Alladi, S., Bak, T. H., Duggirala, V., Surampudi, B., Shailaja, M., Shukla, A. K., Chaudhuri, J. R., & Kaul, S. (2013). **Bilingualism delays age at onset of dementia, independent of education and immigration status.** *Neurology, 81*(22), 1938–1944. https://doi.org/10.1212/01.wnl.0000436620.33155.a4
2. Bialystok, E., Craik, F. I. M., & Luk, G. (2012). **Bilingualism: Consequences for mind and brain.** *Trends in Cognitive Sciences, 16*(4), 240–250. https://doi.org/10.1016/j.tics.2012.03.001
3. Bialystok, E. (2021). **Bilingualism as a contributor to cognitive reserve: What it can do and what it cannot do.** *American Journal of Alzheimer's Disease & Other Dementias.* (Review of bilingualism and cognitive reserve.)
4. Livingston, G., Huntley, J., Sommerlad, A., et al. (2020). **Dementia prevention, intervention, and care: 2020 report of the Lancet Commission.** *The Lancet, 396*(10248), 413–446. https://doi.org/10.1016/S0140-6736(20)30367-6
5. Logan, J. S., Lively, S. E., & Pisoni, D. B. (1991). **Training Japanese listeners to identify English /r/ and /l/: A first report.** *Journal of the Acoustical Society of America, 89*(2), 874–886.
6. Lively, S. E., Logan, J. S., & Pisoni, D. B. (1993). **Training Japanese listeners to identify English /r/ and /l/. II: The role of phonetic environment and talker variability.** *Journal of the Acoustical Society of America, 94*(3), 1242–1255.
7. Mechelli, A., Crinion, J. T., Noppeney, U., O'Doherty, J., Ashburner, J., Frackowiak, R. S., & Price, C. J. (2004). **Neurolinguistics: Structural plasticity in the bilingual brain.** *Nature, 431*, 757. https://doi.org/10.1038/431757a
8. Kraus, N., & Chandrasekaran, B. (2010). **Music training for the development of auditory skills.** *Nature Reviews Neuroscience, 11*(8), 599–605. https://doi.org/10.1038/nrn2882
9. García-Pentón, L., Pérez Fernández, A., Iturria-Medina, Y., Gillon-Dowens, M., & Carreiras, M. (2014). **Anatomical connectivity changes in the bilingual brain.** *NeuroImage, 84*, 495–504.
10. Paap, K. R., & Greenberg, Z. I. (2013). **There is no coherent evidence for a bilingual advantage in executive processing.** *Cognitive Psychology, 66*(2), 232–258.
11. de Bruin, A., Treccani, B., & Della Sala, S. (2015). **Cognitive advantage in bilingualism: An example of publication bias?** *Psychological Science, 26*(1), 99–107.
12. Lehtonen, M., Soveri, A., Laine, A., Järvenpää, J., de Bruin, A., & Antfolk, J. (2018). **Is bilingualism associated with enhanced executive functioning in adults? A meta-analytic review.** *Psychological Bulletin, 144*(4), 394–425.
13. Horwitz, E. K., Horwitz, M. B., & Cope, J. (1986). **Foreign language classroom anxiety.** *The Modern Language Journal, 70*(2), 125–132.

> *Note: reference 3's exact journal/volume should be confirmed against the publisher record before formal citation; the substantive claims it supports (bilingualism as a cognitive-reserve contributor) are corroborated by references 1, 2 and 4.*

---

## 15. Accessibility, credits and licence

**Accessibility.** The interface uses high-contrast neon-on-dark visuals, honours `prefers-reduced-motion`, provides keyboard shortcuts, and presents native scripts in script-appropriate Noto fonts. Full WCAG conformance has **not** been formally audited; comprehensive validation would require testing with assistive technologies and expert accessibility review. Note that a speech-driven game is inherently challenging for users who cannot speak the target phrases; this is an outreach game rather than a certified-accessible product.

**Attribution.** Fonts by the Google Fonts project (Orbitron; Noto families). Mandarin romanisation via `pinyin-pro`. Flags are simplified, hand-authored SVG approximations for stylistic consistency and are not official heraldry. Speech recognition and synthesis are provided by the user's browser.

**Medical disclaimer.** This project is for education and public engagement only. It is not a medical device and does not diagnose, treat or prevent any condition. Nothing here constitutes medical advice.

**Licence.** Add your chosen licence here (e.g. MIT) before publishing.

<div align="center">

*Built with care for learners, players and the curious. Speak the world.*

</div>
