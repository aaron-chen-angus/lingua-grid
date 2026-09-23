# Requirements — LINGUA//GRID

## Introduction
Browser game where players pronounce phrases in 10 languages (English first, then 9 random) to pass neon barriers on a TRON highway. Player details, per-stage results and final times are stored for a leaderboard.

## R1 Player registration
**User story:** As a player, I enter my details so my score appears on the leaderboard.
1. THE SYSTEM SHALL collect Name (1–20 chars, required), Age (integer 5–99, required) and Gender (Male / Female / Prefer not to say, required).
2. THE SYSTEM SHALL display a data-use notice and require a consent checkbox before starting (Singapore PDPA).
3. WHEN any field is invalid THE SYSTEM SHALL block START and highlight the field in orange neon.
4. THE SYSTEM SHALL strip HTML from Name before storing or displaying it.

## R2 Browser and microphone preflight
1. WHEN the browser lacks SpeechRecognition THE SYSTEM SHALL show an "Use Chrome or Edge" screen and not proceed.
2. WHEN the page is not served over HTTPS or localhost THE SYSTEM SHALL show a warning.
3. THE SYSTEM SHALL request microphone permission and show a live level meter; the player SHALL say "ready" (English recognition) to confirm the mic works before the run.
4. WHEN permission is denied THE SYSTEM SHALL show instructions to re-enable it and a RETRY button.
5. WHEN the recognition service reports `network` errors THE SYSTEM SHALL inform the player that an internet connection is required.

## R3 Language pool and selection
1. THE SYSTEM SHALL support: English, Mandarin, Tamil, Hindi, Bahasa Melayu, Bahasa Indonesia, Thai, Burmese, Vietnamese, French, German, Spanish, Hebrew, Arabic, Russian.
2. Stage 1 SHALL always be English. Stages 2–10 SHALL be 9 distinct languages chosen uniformly at random from the remaining eligible languages.
3. A language is eligible only if hint audio is available (bundled MP3 for its phrases OR a `speechSynthesis` voice matching its language code).
4. WHEN fewer than 9 non-English languages are eligible THE SYSTEM SHALL fill remaining stages from `CONFIG.backupLanguages` (Japanese, Korean, Italian, Portuguese, Filipino) that pass the same check, and log which were substituted.
5. WHEN recognition returns `language-not-supported` for a stage THE SYSTEM SHALL mark it `unsupported`, swap in an unused eligible language with no time penalty, and record the swap.
6. For each chosen language THE SYSTEM SHALL pick one random phrase from its phrase bank.

## R4 Barrier presentation
1. Each barrier SHALL display: country flag (SVG), language name, phrase in native script, romanisation (non-Latin scripts), and IPA.
2. Hebrew and Arabic text SHALL render right-to-left.
3. Native script SHALL use fonts that cover the script (Noto Sans SC/Tamil/Devanagari/Thai/Myanmar/Hebrew/Arabic) — never Orbitron.
4. THE SYSTEM SHALL show stage progress (e.g. 03/10), run timer, and penalty total at all times.

## R5 Speech capture
1. THE SYSTEM SHALL start listening on SPACE or clicking the SPEAK button, and stop automatically on end of speech or after `CONFIG.maxListenMs` (default 7000).
2. Recognition SHALL use the stage language's BCP-47 code, `interimResults=true`, `maxAlternatives=5`.
3. WHILE listening THE SYSTEM SHALL show a live waveform and interim transcript.
4. THE SYSTEM SHALL NOT listen while hint audio is playing.

## R6 Pronunciation acceptance
1. THE SYSTEM SHALL compute a similarity score 0–1 between the recognised transcript(s) and the target phrase (and its accepted variants) per design.md.
2. THE SYSTEM SHALL pass the stage WHEN best similarity ≥ the language threshold (default 0.65; Thai, Burmese, Tamil 0.55).
3. THE SYSTEM SHALL display the score as "SYNC nn%" with the recognised text.
4. WHEN similarity ≥ 0.90 THE SYSTEM SHALL show a "PERFECT" callout and apply a 1 s time bonus.
5. Thresholds, bonus and penalties SHALL be editable in `CONFIG`.

## R7 Hints
1. Clicking HINT (or H key) SHALL play the phrase audio in the target language.
2. Each hint use SHALL add 5 s penalty, with an animated "+5s" callout.
3. Replays within the same stage SHALL each cost 5 s.

## R8 Retry and skip
1. On failure THE SYSTEM SHALL show the similarity and allow unlimited retries.
2. After 3 failed attempts on a stage THE SYSTEM SHALL offer SKIP (+15 s penalty); the stage is recorded as skipped.

## R9 Timing and score
1. The run timer SHALL start when barrier 1 appears and stop when barrier 10 is passed.
2. Final time = elapsed + hint penalties + skip penalties − perfect bonuses.
3. Leaderboard ranking SHALL be by final time ascending; ties broken by fewer hints.

## R10 Results and leaderboard
1. THE SYSTEM SHALL show a results screen: final time, rank, hints, skips, perfects, and a per-stage table (flag, language, phrase, heard, SYNC%, attempts, passed/skipped).
2. THE SYSTEM SHALL show a top-10 leaderboard with the current player highlighted.
3. THE SYSTEM SHALL offer PLAY AGAIN (same player) and NEW PLAYER.

## R11 Data storage and export
1. Each run SHALL be stored in `localStorage` per the data model in design.md.
2. An admin panel (Ctrl+Shift+L) SHALL allow CSV export (one row per stage) and JSON export, and clearing data after confirmation.
3. WHEN `CONFIG.leaderboardWebhook` is set THE SYSTEM SHALL POST each completed run as JSON; failures SHALL be queued and retried on next load.

## R12 Aesthetic and feel
1. TRON aesthetic: near-black background, cyan/blue and orange neon, Orbitron UI font, glow on all lines and text.
2. Player's light-cycle rides forward on a perspective grid highway with a light trail; speed lines and parallax skyline.
3. Barriers rise from the road with a scan-line animation; on pass they shatter into particles and the cycle boosts; on fail the barrier flashes orange and shakes.
4. Synthesised SFX for: barrier rise, listening start, pass, perfect, fail, hint, finish.
5. Combo counter for consecutive first-attempt passes.

## R13 Performance and compatibility
1. 60 fps on a typical school laptop at 1920×1080 in Chrome/Edge.
2. Layout usable from 1280×720 to 2560×1440.
3. Initial load < 2 MB excluding optional audio.

## R14 Privacy
1. The notice SHALL state that speech is processed by the browser's cloud speech service and that name/age/gender/scores are stored locally (and sent to the leaderboard webhook if enabled).
2. No audio recordings SHALL be stored.
