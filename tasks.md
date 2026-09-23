# Tasks — LINGUA//GRID

- [ ] 1. Scaffold `index.html`: CONFIG, font links (Orbitron, Noto families), CSS variables for palette, screen containers, CRT overlay. _R12, R13_
- [ ] 2. Inline `PHRASES` from `data/phrases.json` and the language table; build inline SVG flags for all 15 + backup languages. _R3, R4_
- [ ] 3. Implement `Scorer` (normalize, graphemes, levenshtein, windowSim, score) and `tests.html` with the cases in design §11. Load pinyin-pro from pinned CDN. _R6_
- [ ] 4. Implement `Store`: runs CRUD, leaderboard sort, CSV/JSON export, webhook POST + retry queue, admin panel (Ctrl+Shift+L). _R10, R11_
- [ ] 5. Implement BOOT preflight: SpeechRecognition support, HTTPS check, voices load, hint-audio eligibility, language pool build with backup fill. _R2, R3_
- [ ] 6. Registration screen with validation, sanitisation, PDPA notice + consent. _R1, R14_
- [ ] 7. Mic check screen: getUserMedia, level meter, say "ready" test, denied/offline handling. _R2_
- [ ] 8. `Speech.listen()` wrapper with interim transcript, max duration, error mapping, he-IL→iw-IL retry, language-not-supported swap. _R3.5, R5_
- [ ] 9. `Audio`: hint playback (MP3 then speechSynthesis), SFX synth (rise, listen, pass, perfect, fail, hint, finish), block listening during hint. _R7, R12.4_
- [ ] 10. `Renderer`: grid highway, skyline parallax, light-cycle + trail, barrier rise/shatter/shake, particles, speed control; 60 fps. _R12, R13_
- [ ] 11. Barrier card + HUD overlays: flag, name, native text (RTL aware), romanisation, IPA, timer, stage, penalties, combo, SPEAK/HINT/SKIP buttons, SYNC ring, keyboard shortcuts. _R4, R5.3, R8_
- [ ] 12. `Game` state machine wiring: stage sequencing, timing, penalties, perfect bonus, combo, skip after 3 fails, run record assembly. _R8, R9_
- [ ] 13. Results + leaderboard screens, PLAY AGAIN / NEW PLAYER. _R10_
- [ ] 14. Polish pass: transitions, callouts (+5s, PERFECT, COMBO), responsive 1280×720–2560×1440, reduced-motion respect. _R12, R13_
- [ ] 15. Run tests.html; manual checklist per design §11; deploy to GitHub Pages. _All_
