# TEF Vocab Loop — Next Work Roadmap

**Baseline:** v2.6  
**Roadmap principle:** stop adding large surface features temporarily; strengthen learning correctness and long-term reliability first.

---

# Priority 0 — Protect user progress

## P0.1 Upgrade procedure for direct HTML

Until hosted on a stable origin:

- export JSON before opening a new version,
- keep previous stable HTML,
- verify progress after upgrade,
- import backup if necessary.

## P0.2 Add backup health later

- last-backup timestamp
- gentle backup reminder
- import schema validation

---

# Priority 1 — v2.7 Learning Engine Hardening

## 1. Today queue policy

### Current issue

Seen can consume the full review quota and starve overdue normal reviews.

### Proposed decision to discuss

Preferred default:

**Weak due → overdue due → Seen → New**

Alternative:

reserve 25–40% of review capacity for Seen.

### Acceptance criteria

- overdue review cannot be completely starved by a large Seen backlog,
- Seen still continues promptly,
- weak due items remain highest priority,
- new words are last.

---

## 2. Seen backlog controls new-word intake

### Current issue

`effectiveNewCount()` ignores Seen.

### Acceptance criteria

- high Seen backlog automatically reduces or pauses new words,
- home text explains why new count was reduced.

---

## 3. Multi-skill Weak

### Current issue

one `weakSkill` string can be overwritten.

### Target

Track Weak state separately for each core skill.

### Acceptance criteria

- listening and reverse can both be weak,
- recovering listening does not erase reverse weakness,
- card is visibly Weak while any core skill remains weak.

---

## 4. Mastered evidence

### Current issue

one global `longReviewPassed` can be earned by one skill and later unlock Mastered for the whole card.

### Target

Use per-skill or post-Familiar spaced confirmation.

### Acceptance criteria

- Mastered cannot be unlocked by stale long-gap evidence from only one core skill,
- spelling remains excluded from mandatory mastery.

---

## 5. Weak / Hard / Star semantics

Separate:

- Weak = current engine state
- Hard = difficult based on performance
- ★ = user-saved

### Acceptance criteria

- “현재 약점” count means actual current Weak only,
- starred words do not inflate Weak count,
- Hard-focus session labels what it includes.

---

## 6. Due order inside category balancing

Keep category variety but sort each review bucket by due time.

### Acceptance criteria

- very overdue cards are not randomly pushed behind less-overdue cards solely because of balancing.

---

## 7. Daily level scope

Discuss/choose:

- A1-A2
- B1-B2
- Both

### Acceptance criteria

Today new words respect the chosen scope.

---

# Priority 2 — v2.8 Quiz Quality

## 1. Reverse ambiguity guard

Korean prompt must have a reasonably unique expected French answer.

Reject distractors whose meanings overlap too strongly.

For inherently ambiguous cards:

- add context,
- use sentence/cloze,
- or avoid reverse multiple choice.

## 2. Better distractor ranking

Signals:

- same part of speech,
- same category,
- semantic closeness,
- form similarity,
- learner's past confusions,
- ambiguity penalty.

## 3. Spelling real-device test

Test:

- accents
- apostrophes
- œ
- Android IME
- keyboard switching
- autocorrect

Only then decide whether to add special French-character buttons.

## 4. Answer undo

Investigate one-answer transactional undo for accidental taps.

Do not implement by casually decrementing counters; snapshot/restore must be safe.

---

# Priority 3 — v2.9 Word Library / Content QA

## Filters

- New
- Seen
- Learning
- Familiar
- Mastered
- Weak
- Hard
- ★
- ⚑ Review needed

## Sorting

- A–Z
- recently studied
- next review
- weakest
- source order

## Search

Make accent-insensitive.

## Local issue flagging

Add `⚑ 검토 필요`.

Reasons:

- 뜻
- 예문
- 발음
- 문제/보기
- 기타

This is strongly recommended because real studying can become ongoing QA.

---

# Priority 4 — v2.10 Sentence Library Quality

Do not simply expand the number of Recommended items.

Instead improve precision.

## Proposed data fields

- `sentenceRecommended`
- `sentenceTags`
- optional `sentenceValueScore`

Possible tags:

- collocation
- verb+preposition
- connective
- argumentation
- everyday pattern
- TEF reusable
- grammar construction

## Known current scorer problems

False-positive style:

`La feuille tombe de l'arbre.`

False-negative style:

`Elle veut atteindre son objectif avant juin.`

The latter contains a useful collocation but is currently excluded.

## Goal

Recommended should feel like a small reusable phrasebook, not merely a subset selected by numeric heuristics.

---

# Priority 5 — Reliability / Maintainability

## Explicit schema migrations

Separate:

- app version
- data schema version
- session schema version

## Import validation

Reject malformed backups cleanly.

## Regression tests

Automate at least:

- New → Seen
- Seen → Learning
- Familiar
- Mastered
- two simultaneous Weak skills
- Weak recovery
- due-vs-Seen queue
- Seen backlog new-word suppression
- spelling exact/accent/wrong
- session serialize/resume
- category balance
- 2,866 card integrity
- TTS metadata

---

# Priority 6 — Later packaging/public-release work

Only after the learning engine is stable:

- stable HTTPS deployment,
- installable PWA if desired,
- Android WebView/native wrapper if needed,
- login/cloud sync only if the product genuinely needs it,
- streak/ranking only if they improve motivation without distorting learning.

Do not make authentication/ranking the next priority for the current personal app.

---

# Suggested immediate next meeting

Before coding v2.7, decide four product questions:

1. Should overdue reviews be above Seen, or should Seen reserve part of the quota?
2. Should Today study default to A1-A2, B1-B2, or a user-selected scope?
3. What exactly should “Hard” mean compared with Weak and ★?
4. How strict should Mastered be: all three core skills spaced, or one post-Familiar spaced confirmation?

Once these four are decided, v2.7 can be implemented without guessing.
