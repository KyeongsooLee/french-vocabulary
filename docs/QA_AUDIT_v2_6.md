# TEF Vocab Loop v2.6 — Comprehensive QA Audit

**Audit date:** 2026-09-11  
**Baseline:** `TEF_Vocab_Loop_v2_6.html`  
**Scope:** static code QA, data integrity, learning-engine logic review, persistence review, sentence-library review, product/UX risk review.  
**Important limitation:** full browser GUI automation was not available in the current environment. Final touch/UI behavior still needs real Android testing.

---

# 1. Executive summary

The project is in a healthy state for a personal beta.

The following core areas are working and internally consistent:

- 2,866-card corpus integrity,
- separate meaning/listening/reverse/spelling tracking,
- Seen vs Learning distinction,
- TTS/display separation,
- category-balanced custom study,
- previous-introduction replay,
- resumable sessions,
- word-library detail view,
- sentence library,
- offline/no-network architecture,
- spelling logic,
- JSON backup/restore foundation.

No syntax-breaking JavaScript error or missing core-card-field problem was found.

However, the audit found several **important learning-engine issues that should be fixed before adding many more features**:

1. **Seen items can starve overdue reviews in Today study.**
2. **A large Seen backlog does not reduce the number of new words introduced.**
3. **Only one `weakSkill` can exist at a time; weakness in a second skill overwrites the first.**
4. **Mastered uses one global `longReviewPassed` flag, which can overstate long-term mastery of other core skills.**
5. **“현재 약점” currently counts starred cards and historically difficult cards, not only current Weak cards.**
6. **The Today button has no study-level scope and can introduce A1-A2 and B1-B2 together.**
7. **Reverse Korean→French multiple choice can become semantically unfair when several French words are valid translations of the Korean prompt.**
8. **Sentence recommendation is useful but still only a heuristic; it has identifiable false positives and false negatives.**
9. **Direct-open `file://` updates have a browser-storage portability risk across renamed HTML files.**
10. **State/schema migration should become explicit before the project grows further.**

Recommended next release: **v2.7 = learning-engine hardening**, not another large feature.

---

# 2. Automated/static checks that passed

## JavaScript

- `node --check`: **PASS**
- No static duplicate HTML IDs detected.
- All dynamic selector IDs referenced in JavaScript were found in generated template markup.

## Corpus

- Total cards: **2,866**
- Missing `fr`: 0
- Missing `ko`: 0
- Missing `tts`: 0
- Missing `exFr`: 0
- Missing `exKo`: 0
- Missing `level`: 0
- Missing `category`: 0
- Duplicate card IDs: 0

## Confusable references (`cf`)

- Every card has 12 `cf` references.
- Invalid referenced IDs: 0
- Self-references: 0
- Same-level references: 100%
- Same-category references: about 99.76%

This is structurally strong. Semantic quality is discussed later.

## TTS

- Known grammar metadata patterns remaining in `tts`: 0
- Stray trailing `f` / `m` metadata artifacts: 0

The display/TTS separation remains intact.

## Offline/privacy architecture

- No `fetch()` calls found.
- No external HTTP/HTTPS URLs found.
- No external network dependency detected in the HTML.

That is a strong property for the current personal/offline workflow.

---

# 3. Corpus findings that are not necessarily bugs

Four same-level French surface forms occur on more than one card, but they represent different meanings/categories and should not be blindly deduplicated:

- `tendre`
  - verb: 당기다, 내밀다
  - adjective/personality: 부드러운, 상냥한
- `la société`
  - 회사
  - 사회, 회사
- `la démonstration`
  - 증명, 증거
  - 시위, 논증, 증거
- `procureur`
  - 검사
  - 검사, 공판관

These are semantic/polysemy cases, not simple technical duplicate IDs.

Any future deduplication pass must compare meaning/category, not only `fr`.

---

# 4. HIGH PRIORITY finding: Today queue can starve overdue reviews

Current Today construction is effectively:

1. Weak + due
2. Seen
3. normal due
4. slice to review limit
5. add new words

The important problem is that **Seen can consume the review quota before ordinary overdue reviews**.

Example:

- maximum review = 40
- current Seen = 40
- overdue normal reviews = 100
- no Weak cards

Current ordering can select:

- 40 Seen
- 0 overdue normal reviews
- then still add new words

This is not ideal spaced-repetition behavior.

### Why it matters

A Seen item is unfinished, but an overdue review is a time-sensitive memory event.

Seen deserves priority, but it should not be able to indefinitely starve all scheduled reviews.

### Recommended fix

Use a policy such as:

**Weak due → overdue due → Seen continuation → New**

or reserve quotas/interleave:

- at least 60–70% of review capacity for due reviews,
- up to 30–40% for Seen continuation,
- then new words only if backlog is under control.

This decision should be made deliberately in v2.7.

---

# 5. HIGH PRIORITY finding: Seen backlog does not reduce new-word intake

`effectiveNewCount()` currently looks at **due review count**, but not Seen backlog.

Example:

- Seen = 100
- due = 0
- dailyReview = 40
- dailyNew = 20

Current logic can still produce:

- 40 Seen
- + 20 New

That can create more unfinished Seen cards while a large Seen backlog already exists.

### Recommended fix

New-word load should respond to:

- due backlog,
- Seen backlog,
- possibly active unfinished session backlog.

Possible policy:

- Seen >= dailyReview → 0 new
- Seen >= dailyReview / 2 → reduce new by 50%
- otherwise normal new quota

This directly supports the project principle:

> unfinished learning should be completed before continuously adding more exposure.

---

# 6. HIGH PRIORITY finding: only one Weak skill can exist

Current progress stores:

`weakSkill: <one skill or null>`

That means a card cannot simultaneously be weak in:

- listening
- and reverse

If listening is Weak and repeated reverse failures occur, `weakSkill` can be overwritten from `listening` to `reverse`.

Later, recovery of reverse can clear the card's Weak state even though listening may still be unstable.

### This conflicts with an earlier project principle

The engine intentionally tracks skills independently.

Weakness should therefore also be independently representable.

### Recommended structure

Replace conceptually:

`weakSkill: "listening"`

with something like:

`weakSkills: { listening: {...}, reverse: {...} }`

or a set/map.

Each skill should have its own:

- weakSince
- recovery count
- last recovery success

The visible card state can remain `Weak` if any core skill is weak.

This is an important v2.7 engine fix.

---

# 7. HIGH PRIORITY finding: Mastered long-gap evidence is global, not per skill

Current card progress contains one boolean:

`longReviewPassed`

It becomes true when **any one core skill** has a sufficiently long success gap.

Later, if all core skills reach the required step, the card can become Mastered because the old global boolean is already true.

### Example failure mode

1. Meaning gets a long-gap success and sets `longReviewPassed=true`.
2. Listening/reverse are still weak.
3. Much later, listening/reverse are raised to step 3 in a short period.
4. Card becomes Mastered immediately because the old meaning long-gap flag is still true.

That is weaker evidence than the product philosophy intended.

### Recommended fix

Track long-term evidence per core skill:

- `meaning.longReviewPassed`
- `listening.longReviewPassed`
- `reverse.longReviewPassed`

Then define Mastered using an explicit rule, e.g.:

- all core skills step >= 3,
- and either all three have spaced evidence,
- or at minimum the card receives one post-Familiar spaced confirmation after all core skills reached the threshold.

The exact rule should be chosen before coding.

---

# 8. HIGH/MEDIUM finding: “현재 약점” does not mean only current Weak

`isHardCard()` returns true when:

- starred manually,
- current `weakSkill` exists,
- or historical cumulative error rate is >= 35% after enough attempts.

Home currently uses this for the label:

**현재 약점**

Therefore:

- a manually starred word can increase “현재 약점,”
- a word that was difficult months ago can remain counted,
- current Weak and historical Hard are mixed.

### Recommendation

Separate three concepts:

- **Weak** = current unstable learning state
- **Hard** = historically difficult / high recent error tendency
- **★** = manually saved by user

UI options:

- Home: `Weak` only for “현재 약점”
- Library/filter: separate `어려운 단어` and `★`
- Weak-focused session can optionally include all three, but label it clearly.

---

# 9. HIGH/MEDIUM product issue: Today study has no active-level scope

Custom study supports:

- A1-A2
- B1-B2
- All

But `todayTargets()` draws new words from the entire 2,866-card corpus.

Therefore Today study can introduce B1-B2 before the user has intentionally chosen to work on that level.

This may be correct if the desired product is a mixed full-corpus trainer, but the app currently has no explicit setting that tells the engine this is intended.

### Recommended decision

Add a **Daily study scope** setting:

- A1-A2 only
- B1-B2 only
- Both

Possibly later:

- selected categories only

This should be decided with the user rather than silently changing behavior.

---

# 10. Quiz-quality finding: reverse questions can be semantically ambiguous

The corpus contains many Korean meanings shared by multiple French words.

Audit found:

- 91 exact normalized Korean meanings are used by multiple cards,
- involving 190 cards.

Examples include conceptually related/synonymous items such as:

- `parler` / `dire`
- `regarder` / `voir`
- `soudain` / `soudainement` / `tout à coup` / `brusquement`
- `savoir` / `connaître`

The direct `cf` lists appear to avoid **exact identical Korean meanings** among direct confusable pairs, which is good.

However, reverse mode still asks:

> Korean meaning → one specific French answer

For broad/synonymous Korean definitions, more than one French choice may be linguistically reasonable even if only one card is marked correct.

### Recommendation

Before making distractors harder, add an **ambiguity guard**.

For reverse questions:

- do not use distractors with strongly overlapping Korean definitions,
- or use a short context/example clue,
- or convert high-ambiguity cards to cloze/context questions,
- or support multiple accepted answers only when they are truly equivalent.

A “harder distractor” is not better if it makes the question unfair.

---

# 11. Distractor quality is structurally good but semantically uneven

Positive:

- 12 confusable refs per card,
- almost all same category,
- all same level.

But manual sampling found mixed semantic quality.

Good examples:

- `l'oxygène` → `dioxyde de carbone`, `charbon`, environmental terms
- `la chemise` → tee-shirt, veste, robe, jupe

Weaker examples:

- `toujours` had distractors such as `normal`, `usuel`, `pressé`
- some verb groups are same category but not genuinely confusable

### Recommended next-generation distractor score

Use several signals:

1. same level
2. same category
3. same part of speech / grammatical shape
4. Korean semantic similarity
5. French form similarity where pedagogically relevant
6. user error history
7. ambiguity exclusion

This should be a v2.8 task after core engine hardening.

---

# 12. Sentence recommendation QA: useful, but scorer needs a second generation

Current Recommended set:

- 353 / 2,866
- A1-A2: 78
- B1-B2: 275
- curated: 269
- reviewed: 84

This is intentionally selective and works as a first version.

However, a deeper audit shows the scoring rule is not yet perfectly aligned with the intended definition:

> “worth studying as a reusable sentence.”

### Finding A: probable false positives

Of the 353 recommended examples, about **191** reached the threshold without a direct strong signal from the current audit categories such as:

- known useful construction,
- connective,
- modal/argumentative pattern,
- infinitive complement,
- explicit grammar metadata.

Some are still good sentences, but recommendation may be coming mostly from:

- curated status,
- B1-B2 level,
- category bonus,
- sentence length.

Example currently recommended:

`La feuille tombe de l'arbre.`

It is a valid example, but its reusable sentence-learning value is limited.

### Finding B: false negative

A clearly useful collocation example:

`Elle veut atteindre son objectif avant juin.`

for `atteindre`

currently scores below the recommendation threshold and is excluded.

Yet `atteindre un/son objectif` is exactly the kind of reusable collocation the Sentence tab was intended to surface.

### Recommended fix

Do not rely only on dynamic regex scoring.

Better approach:

- create a stable per-card field such as `sentenceValue`,
- use automated scoring to generate candidates,
- then review/confirm the recommended set,
- explicitly tag:
  - collocation
  - verb + preposition
  - reusable structure
  - TEF argumentation
  - connective
  - useful everyday pattern

The final recommendation set can still be ~300–600 items, but it should be curated around pedagogical value.

---

# 13. Example-library data observations

Duplicate French example strings:

- 14 duplicated sentence strings,
- 28 cards affected.

This is not automatically wrong; some sentences legitimately demonstrate different related cards.

Examples include:

- `Elle doit rédiger un rapport pour demain.`
- `Il tend la main pour m'aider.`

Two duplicated sentences also appear in the current Recommended set.

This is low severity, but the Sentence library may feel more varied if exact duplicate sentence cards are deduplicated visually or linked to multiple words.

---

# 14. Generic example inventory

The corpus still contains many intentionally simple word-demonstration examples.

Audit counts include approximately:

- 209 examples matching a very simple `C'est X.` style pattern
- 34 examples matching `On parle souvent de ... dans les médias.`

Most are template examples and are not necessarily incorrect.

Important distinction:

- **word-card example quality** can be acceptable,
- **Sentence-tab recommendation quality** should be stricter.

Do not rewrite all simple examples merely to make every sentence sophisticated.

---

# 15. Search UX blind spot: accents

Word and sentence search currently use lowercase substring matching.

A learner may type:

- `eleve`
- while the data contains `élève`

and expect it to match.

### Recommendation

Use accent-insensitive normalization for search.

The app already has normalization logic that can be adapted.

This is a small, high-value usability improvement.

---

# 16. Word-library features still missing

The current Word tab is useful, but a real library would benefit from:

### Filters

- New
- Seen
- Learning
- Familiar
- Mastered
- Weak
- ★
- Hard

### Sorting

- source order
- French alphabetical
- recently studied
- next review
- weakest first

### Per-card maintenance

Useful future controls:

- reset only this word's progress,
- flag a suspicious meaning/example/TTS item,
- possibly “known already” later, if carefully designed.

Do not add “mark mastered” casually because it bypasses learning evidence.

---

# 17. Missing user-feedback loop for content errors

The user is actively studying the corpus and is therefore the best QA source.

Currently, when the user notices:

- strange Korean meaning,
- awkward example,
- bad TTS,
- ambiguous quiz,
- poor distractor,

there is no in-app way to mark the card.

### Recommended feature

Add a local flag:

`⚑ 검토 필요`

Optional reasons:

- 뜻
- 예문
- 발음
- 문제/보기
- 기타

Then add a library filter:

`검토 필요`

This would turn normal studying into continuous content QA and make future revision much easier.

This is highly recommended.

---

# 18. Accidental answer taps are still not undoable

The app now protects accidental new-word advancement with previous-introduction replay.

But if the learner accidentally taps a wrong multiple-choice option, that wrong answer immediately affects:

- attempts,
- error history,
- retry scheduling,
- possibly Weak logic.

### Possible future feature

A one-step:

`방금 답변 취소`

This is more complex than intro replay because it requires a transactional snapshot of:

- card progress,
- session counters,
- pending tasks.

It should only be added if implemented safely.

Not urgent, but the same real-world mistap problem that motivated intro back can happen here too.

---

# 19. Spelling mobile UX still needs real-device QA

Static spelling logic passes.

Remaining real-device questions:

- Korean/English/French keyboard switching friction
- accented letter entry
- `œ`
- apostrophe behavior
- autocorrect interference
- IME composition behavior
- Enter key behavior on Android

Potential future helper:

small optional French-character buttons:

`é è ê ë à â î ï ô ù û ü ç œ`

Do not add them unless actual phone use shows they are needed.

---

# 20. Update/storage portability risk with local HTML files

The app's database name/key is stable, so progress should carry when browser origin/storage identity stays the same.

However, the app is opened from downloaded local HTML files.

Browser behavior for `file://` storage can be implementation-dependent.

If the user downloads:

- `v2_5.html`
- then `v2_6.html`

the browser may not always treat both files as the same persistent storage origin.

### Risk

A new version can appear to have no progress even though the old file still has it.

### Recommended current workflow

Before changing versions:

1. export JSON backup from the old version,
2. open new version,
3. verify progress,
4. import backup if needed,
5. keep previous stable HTML until confirmed.

### Longer-term fix

When convenient:

- host on a stable HTTPS origin (e.g. GitHub Pages),
- or use a native/WebView wrapper with controlled storage.

This is not a reason to abandon direct HTML now, but it should be treated as a known operational risk.

---

# 21. Persistence/schema architecture should mature now

The app uses:

- app/engine version,
- `dataVersion: 2`,
- `sessionVersion: 1`,
- normalization for older state.

The product has now gained:

- Seen,
- active sessions,
- sentence library,
- richer progress behavior.

Before more schema changes, introduce an explicit migration model.

Recommended:

- `APP_VERSION`
- `DATA_SCHEMA_VERSION`
- `SESSION_SCHEMA_VERSION`

and migration functions:

- `migrateDataV2toV3`
- `migrateSessionV1toV2`

Do not rely indefinitely on ad hoc field normalization.

---

# 22. Backup import validation is permissive

Import currently verifies JSON parsing, but a malformed object with the expected `dataVersion` may still be accepted before deeper errors appear.

### Recommended

Validate:

- progress is an object,
- card IDs exist,
- skill fields are sane,
- settings are sane,
- activeSession IDs are valid,
- schema version is supported.

After import, show:

- cards with progress,
- active session status,
- backup version.

This reduces the chance of silently importing corrupted state.

---

# 23. Backup reminders are worth adding

Browser-local storage is not a permanent backup.

For a long-term 2,866-word learning project, a user could lose months of progress if:

- browser data is cleared,
- phone changes,
- file-origin storage changes,
- app data is reset.

### Recommended

After every N sessions or every 7–14 days:

> `최근 백업이 없습니다. 학습 기록을 내보낼까요?`

Keep it non-annoying and dismissible.

---

# 24. Recommendation-score performance

Sentence recommendation score is recomputed using many regexes during filtering/sorting.

2,866 cards is small enough that it is currently workable, but on a slower phone repeated search input may cause unnecessary work.

### Easy optimization

Cache once:

- `exampleScore`
- `recommended`

in a `Map`, or precompute fields in the card data.

Low priority unless the user notices lag.

---

# 25. Custom-study due ordering has a subtle priority issue

Category balancing uses different behavior based on rank.

With current rank values:

- Weak: 0
- Seen: 1
- Due: 2
- New: 3

The balancing helper sorts buckets by due time only when `rankValue <= 1`.

That means ordinary Due cards (`rankValue = 2`) are shuffled inside category buckets instead of keeping oldest/most overdue first.

### Recommended fix

For review-like buckets, including Due, sort by due time inside each category before round-robin balancing.

This preserves both:

- category variety,
- overdue priority.

---

# 26. Random comparator technical debt

In a selected single category, equal-ranked cards use:

`Math.random() - .5`

inside `sort()` as a tiebreaker.

This is a common shortcut but is not a proper uniform shuffle and can behave inconsistently.

The project already has a correct `shuffle()` helper.

### Recommended

Group by priority, then use `shuffle()` explicitly.

Low severity, easy cleanup.

---

# 27. Home progress semantics could be clearer

The large overall progress count is effectively:

`all cards except New`

So Seen cards contribute to the percentage.

That is legitimate as **exposure progress**, but a learner may read the number as “learned words.”

### Recommendation

Clarify label, e.g.:

- `본 단어`
- `노출된 단어`

and optionally show a second number later:

- `Familiar + Mastered`

This avoids overclaiming learning progress.

---

# 28. Session result metrics could be more explicit

Current result screen includes:

- accuracy,
- new words,
- review words,
- time,
- weakest area,
- current weak cards,
- next-day estimate.

Potential ambiguity:

“새 단어 12” means words included/handled in the session, not necessarily mastered or even successfully recalled.

Suggested wording:

- `새 단어 학습`
- or `새 단어 다룸`

Optional future metrics:

- newly Seen
- moved to Learning
- moved to Familiar
- recovered from Weak

Not urgent.

---

# 29. Accessibility / visual QA not yet complete

Static code cannot fully judge:

- text contrast in sunlight,
- 9–10px labels on small Android screens,
- touch target size,
- screen-reader usability,
- landscape layout,
- system font-size scaling,
- very long French/Korean strings.

Recommended manual QA later:

- smallest phone width used by the user,
- 125–150% system text scaling,
- long B1-B2 expressions,
- keyboard-open spelling layout.

---

# 30. What should NOT be changed casually

The audit does not recommend undoing these successful decisions:

- single HTML direct-open workflow,
- Seen state,
- skill independence,
- spelling non-mandatory mastery,
- delayed retry,
- category-balanced whole-topic study,
- safe intro replay,
- word/example audio visual separation,
- bottom-sheet word library,
- sentence Recommended/All split,
- local/offline architecture.

The next work should harden these systems, not replace them.

---

# 31. Recommended next release sequence

## v2.7 — Learning engine hardening

Must address:

1. Today queue fairness: Due vs Seen.
2. Seen backlog affects new-word quota.
3. multi-skill Weak tracking.
4. stronger Mastered spacing logic.
5. distinguish Weak / Hard / ★.
6. preserve due-time order during category balancing.
7. decide Daily study level scope.

Do not add many new UI features in this release.

## v2.8 — Quiz quality

1. better semantic distractors,
2. reverse-question ambiguity guard,
3. context-based reverse questions where necessary,
4. investigate real mobile spelling experience,
5. optionally transactional answer undo.

## v2.9 — Library and content QA

1. state/star/hard filters,
2. accent-insensitive search,
3. sorting,
4. `⚑ 검토 필요`,
5. content review workflow.

## v2.10 — Sentence Library quality

1. second-generation recommendation model,
2. stable per-card sentence-value tags,
3. fix known false positives/false negatives,
4. optional sentence favorites,
5. deduplicate duplicate sentence display where useful.

## Before wider/public distribution

1. explicit schema migrations,
2. backup validation,
3. backup reminder,
4. stable hosted origin or controlled app storage,
5. accessibility/mobile QA,
6. privacy/about/version screen,
7. regression test suite.

---

# 32. QA conclusion

The app is already much more coherent than a simple vocabulary demo.

The biggest remaining risks are no longer “can the app do X?” but:

- **does the scheduling policy protect long-term memory?**
- **does each status mean exactly what the user thinks it means?**
- **can one skill's behavior accidentally contaminate another?**
- **can progress survive months of real use and upgrades?**
- **are harder questions actually fair?**

That is a healthy sign: the project has moved from feature construction into product/learning-engine quality work.
