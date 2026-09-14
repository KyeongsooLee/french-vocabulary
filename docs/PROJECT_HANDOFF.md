# TEF Vocab Loop — PROJECT HANDOFF

**Current baseline:** `TEF_Vocab_Loop_v2_7.html`  
**Read first in a new session.** For the full reasoning/history, read `TEF_Vocab_Project_History.md`.

---

## 1. What this project is

A Korean-user-focused French vocabulary learning app built as a single offline-friendly HTML file.

Primary goals:

- learn a 2,866-card A1-A2/B1-B2 French vocabulary corpus,
- support pronunciation-heavy learning,
- distinguish different kinds of knowledge,
- schedule delayed review,
- work smoothly on Android,
- preserve progress locally,
- stay focused rather than becoming a general language-learning platform.

The user prefers a modern, uncluttered UI and gives detailed real-use feedback.

---

## 2. Static corpus

- 2,866 unique vocabulary cards.
- A1-A2: 1,465.
- B1-B2: 1,401.
- Source rows before within-level duplicate collapse: 2,917.
- Korean source definitions are preserved as source data.
- Every card has example fields and a TTS-safe pronunciation field.

Typical card fields:

- `id`
- `level`
- `category`
- `fr`
- `ko`
- `pages`
- `occurrences`
- `cf`
- `tts`
- `exFr`
- `exKo`
- `exKind`

Do not change IDs casually; progress is keyed by card ID.

---

## 3. Learning-state model

Visible states:

**New → Seen → Learning → Familiar → Mastered**

plus **Weak** as current instability.

Definitions:

- **New**: never introduced.
- **Seen**: introduction viewed, but no real core quiz attempt.
- **Learning**: real learning has begun.
- **Familiar**: stronger core-skill evidence.
- **Mastered**: stronger spaced evidence, including long-interval success.
- **Weak**: recent repeated instability; recoverable.

Key rule:

> **Seeing is not learning.**

`확인했어요` alone must not count as Learning.

---

## 4. Skills

Tracked independently:

- meaning
- listening
- reverse
- spelling

Spelling is reinforcement and **not a mandatory mastery gate**.

Errors should affect the relevant skill, not wipe out the entire card.

---

## 5. Scheduling principles

- New word is introduced before testing.
- First meaning test appears after intervening questions.
- Errors reappear after several intervening questions.
- Pending retry duplication is prevented.
- Same-day repeated success should not over-promote a card.
- Spaced success matters more than same-session repetition.
- Review intervals broadly follow same-session → 1d → 3d → 7d → 14d → 30d → 60d.

---

## 6. Study selection

Whole-topic custom study intentionally uses **category-balanced selection**.

Reason: earlier code effectively selected the first N cards after priority sorting, which reflected PDF/source order and caused verb-heavy sessions.

Current intent:

- preserve weak/review/new priorities,
- balance categories within the relevant pool,
- respect an explicitly selected category.

Seen unfinished items should receive high continuation priority.

---

## 7. New-word back behavior

The back arrow after an accidental `확인했어요` is a **safe previous-introduction replay**.

It does not roll back progress/scheduling.

Reason: the UX need is “show me the word I accidentally skipped,” not “reverse the engine.”

---

## 8. Answer feedback layout invariant

Final intended visual hierarchy:

- answer feedback background
  - result
  - word/meaning/status
  - word audio / slow word audio
- separate white example card
  - French example
  - Korean example
  - sentence audio / slow sentence audio
- Next

Do not put word-audio controls inside the white example card again.

The user explicitly rejected that merged-background look.

Also keep the redundant system-like explanatory sentence removed.

---

## 9. Word library

Word tab is a personal vocabulary library.

Tapping a word opens a bottom sheet containing:

- word/meaning,
- level/category/status,
- word TTS,
- example + translation,
- example TTS,
- skill bars,
- last study,
- next review,
- answer stats,
- star/hard toggle.

PDF source/page is intentionally not shown in the UI.

---

## 10. Sentence library

Bottom tabs:

**오늘 · 학습 · 단어 · 문장 · 설정**

Sentence tab supports:

- Recommended / All,
- search,
- level filter,
- category filter,
- normal/slow sentence TTS,
- jump to linked word detail.

Recommended is intentionally selective.

At v2.6 QA:

- 353 / 2,866 examples recommended
- A1-A2: 78
- B1-B2: 275
- curated: 269
- reviewed: 84

Interpretation: these are sentences worth studying **as sentences**, not the only acceptable examples in the app.

---

## 11. TTS rule

Display and speech text are separate.

Example:

- display: `l'expression (f.)`
- TTS: `l'expression`

Do not blindly strip all parentheses because some parentheses contain real lexical variants.

---

## 12. Session persistence

Active study sessions are resumable.

Home can show:

- progress,
- source/type,
- continue,
- start over.

Exiting study saves the current session.

Completed sessions clear the saved active session.

Long closed-app time should not count as study duration.

---

## 13. Spelling

Current QA found 2,166 / 2,866 cards eligible under the simple-form spelling rule.

Behavior includes:

- exact accepted,
- accent-only differences can be “almost correct,”
- early leniency in some article cases,
- stricter later checking,
- complex multi-form cards excluded.

Spelling remains non-mandatory for mastery.

---

## 14. Persistence and portability

The single HTML contains:

- app logic,
- static corpus,
- examples.

Learner progress is stored in browser storage.

For development continuity:
- HTML is enough.

For learner-progress continuity:
- HTML + exported JSON backup is needed.

---

## 15. QA expectations

The environment has not reliably supported full GUI browser automation.

Use:

- JS syntax checks,
- card-count assertions,
- data integrity checks,
- targeted logic tests,
- learning-state simulations.

Then rely on real Android testing for final visual/mobile behavior.

---

## 16. Do not break these

- 2,866 stable card identities.
- Seen ≠ Learning.
- skill independence.
- spelling not required for mastery.
- delayed retry.
- category-balanced whole-topic study.
- safe intro replay.
- resumable session.
- word/example audio visual separation.
- TTS display/speech separation.
- direct-open Android HTML workflow.
- existing progress compatibility.

---

## 17. Current likely next work

High-value candidates:

1. Better distractor quality.
2. Word-library state filters.
3. Review generic/template sentence quality.
4. Later: selective cloze questions from high-value examples.

Before implementing any of these, read the relevant sections of the full history.

---

## 18. New-session instruction

When continuing in a new GPT session, say:

> This is an existing project. Read `PROJECT_HANDOFF.md` first, then `TEF_Vocab_Project_History.md`, then inspect the HTML. Preserve the documented product decisions unless I explicitly ask to change them. Before modifying code, identify which existing decisions the change touches. After modifying, update the documentation and QA notes.
---

## 19. Post-v2.6 comprehensive audit

A deeper audit identified several items that should be handled before major new features.

Read `QA_AUDIT_v2_6.md` and `ROADMAP_NEXT.md`.

Highest priority:

1. prevent Seen from starving overdue reviews,
2. make Seen backlog reduce new-word intake,
3. replace single `weakSkill` with multi-skill weakness,
4. strengthen Mastered spaced-evidence logic,
5. separate Weak / Hard / ★ semantics,
6. preserve due-date priority inside category balancing,
7. decide Today level scope.

The recommended next release is **v2.7 Learning Engine Hardening**.
---

## 20. v2.7 Smart Session direction

The user decided that fixed daily counts do not match real use because study volume can vary greatly by day.

A Smart Session design was simulated before implementation.

Read:

`SMART_SESSION_DESIGN_v2_7.md`

Decision:

- Today study should become dynamic/continuous.
- The engine should choose the next task after every interaction.
- Fixed daily new/review caps should no longer be the central scheduler.
- Custom study remains fixed-size.
- New-word pace should adapt to Due/Seen/Weak backlog.
- Implementation was completed in `TEF_Vocab_Loop_v2_7.html`.
---

## 21. v2.7 implementation contract

A detailed implementation contract now exists:

`V2_7_IMPLEMENTATION_SPEC.md`

Most important rule:

> Today Study and Custom Study are two different session types operating on one shared global learner record.

Session queues/retries must be revalidated against latest global progress when resumed, because the other mode may have changed the same card.

Recommended release split:
- v2.7: Smart Session + Today/Custom synchronization
- v2.8: multi-skill Weak + Mastered redesign
- v2.9: distractor/ambiguity quality

This split is safer than changing all core learning-state logic at once.
---

## 22. v2.7 implemented baseline

`TEF_Vocab_Loop_v2_7.html` is now the current implementation baseline.

Today Study:
- continuous Smart Session,
- no fixed daily completion size,
- dynamic next-task selection,
- adaptive New pacing.

Custom Study:
- remains finite,
- retains level/category/size/focus controls.

Session storage:
- Today and Custom can coexist in separate session slots.
- Both write to one shared global learner record.
- saved tasks are revalidated against latest card progress on resume.

Important v2.7 safety behavior:
- v2.6 fixed Today queue is retired during migration,
- card-level progress remains,
- finite Custom session can be retained,
- New/Retry delays in Smart use real intervening interactions,
- Weak cards have a Smart-session anti-spam cooldown,
- Smart daily visible counters reset across calendar days.

Read `V2_7_QA_REPORT.md` before changing the Smart scheduler.

Still deferred:
- multi-skill Weak
- Mastered evidence redesign
- Weak/Hard/★ separation
- distractor/ambiguity work
---

## 22. v2.7.1 automatic spelling rule

Real-device testing found that v2.7 automatic/mix study never produced spelling.

Fixed in v2.7.1.

Important distinction:
- `CORE` remains meaning/listening/reverse for mastery logic.
- spelling remains optional for mastery.
- automatic practice MUST still include spelling as reinforcement for eligible simple French forms.

Do not remove spelling from auto practice merely because it is not a mastery gate.

---

## 23. v2.7.2 number-display rule

Actual French number-value cards display Arabic numerals in the UI.

Example:
- `vingt` → `20`, not `스물`
- `cent` → `100`, not `백`

The original PDF Korean definitions remain stored unchanged.

Only true numeric values use the numeral presentation layer; `le nombre`, `le numéro`, `le chiffre`, etc. are ordinary vocabulary and keep Korean meanings.

Numeric answer choices should stay numeric as well.

---

## 24. v2.7.3 number-display rule

Number presentation is intentionally split:

- **1–99:** Arabic numerals
- **100 and above:** original Korean number words (`백/천/만/억...`)

Do not convert large values back to long Arabic-number strings unless the user explicitly asks.

---

## 25. v2.8.0 bilingual invariant

Current baseline: `TEF_Vocab_Loop_v2_8_0.html`.

The app supports Korean and English study/interface modes.

Do not break these rules:
- one French card corpus,
- one shared learner-progress record,
- original `ko` and `exKo` source data remain intact,
- English content is a sidecar keyed by the same card IDs,
- 1–99 true number cards use Arabic numerals in both modes,
- 100+ number cards use Korean wording in Korean mode and English wording in English mode,
- category values remain original source strings; visible labels are localized,
- an unanswered quiz rerenders after language switching.

English content is complete structurally but bulk-generated. Do not claim every English line was individually human-reviewed.

---

## 26. v2.8.1 bottom-nav size invariant

The compact v2.7-style bottom navigation is intentional.

- icon: 20px
- label: original 10px button text size

Do not apply the icon span font-size rule to translated label spans.

---

## 27. v2.8.2 spelling/TTS invariants

Spelling:
- no `_ _ _` letter-grid display,
- normal continuous text input,
- hint is opt-in,
- hint reveals only the beginning,
- hinted correct answers count as correct but do not advance spelling strength.

TTS:
- never speak grammar shorthand such as `+ inf`, `+ ind`, `+ sub`, `+ cond`, `qn`, `qc`.
- display notation and spoken text remain separate concerns.

---

## 28. v2.8.3 streak calendar

The header streak pill is intentionally clickable.

It opens a month calendar based on the existing `state.studyDays` history:
- purple fill = studied day,
- purple outline = today,
- previous/next month navigation,
- current streak + visible-month study-day count.

Do not create a separate calendar progress store.

---

## 29. v2.8.4 responsive-calendar invariant

The calendar grid cell and the visible date circle are intentionally separate.

- `.calendarDay` = seven-column layout cell
- `.calendarDate` = bounded visual marker
- never restore `aspect-ratio:1/1` on the whole grid cell
- keep `repeat(7,minmax(0,1fr))`
- keep viewport-aware modal max-height/overflow protections

This was required by real Android testing.

---

## 30. v2.8.5 today-calendar invariant

Calendar semantics are intentionally separated:

- purple fill = studied day,
- warm-colored date number = today,
- today before study = hollow neutral circle,
- today after study = purple filled circle but the date number keeps the today accent.

Do not use purple outline alone to mean today.

---

## 31. v2.8.6 calendar-class invariant

Do not use the generic class `today` on calendar date cells.

The Home screen already uses `.today` for its layout.

Calendar current-day state must use:
- `.calendarDay.isToday`

This prevents mobile layout CSS from shifting the current date.

---

## 32. v2.9.0 example-quality baseline

Current baseline: `TEF_Vocab_Loop_v2_9_0.html`.

A staged example-quality rewrite has begun.

v2.9.0 changes 286 low-value A1-A2 examples and synchronizes:
- `exFr`
- `exKo`
- English sidecar example `e`

Do not mass-regenerate the remaining corpus blindly.

Next content passes should prioritize:
1. B1-B2 `L’adjectif` `C'est ...` placeholders,
2. `On parle souvent de ... dans les médias`,
3. repetitive location templates,
4. repetitive health/history/society templates.

Use `EXAMPLE_UPGRADE_v2_9.tsv` to audit this pass.

---

## 33. v2.9.1 example-quality baseline

Current baseline: `TEF_Vocab_Loop_v2_9_1.html`.

Example-quality work now includes:
- v2.9.0: 286 rewrites
- v2.9.1: 167 additional rewrites

Major removed template families:
- mass `J'aime ...`
- B1-B2 adjective bare `C'est ...`
- `On parle souvent de ... dans les médias.`
- `Nous passons près de ...`

Continue staged review rather than regenerating all remaining examples at once.

---

## 34. v2.9.2 example-quality baseline

Current baseline: `TEF_Vocab_Loop_v2_9_2.html`.

Cumulative staged rewrite counts:
- v2.9.0: 286
- v2.9.1: 167
- v2.9.2: 159
- cumulative: **612 examples**

Removed major low-value families now include:
- `J'aime ...`
- B1-B2 bare adjective `C'est ...`
- `On parle souvent de ... dans les médias`
- `Nous passons près de / du ...`
- `Je parle souvent avec ...`
- `Le médecin examine ...`
- `On voit souvent ... dans la nature`

Next recommended bulk pass:
environment + news + religion + history generic frames.

