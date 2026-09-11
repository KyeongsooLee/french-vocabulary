# TEF Vocab Loop — PROJECT HANDOFF

**Current baseline:** `TEF_Vocab_Loop_v2_6.html`  
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
