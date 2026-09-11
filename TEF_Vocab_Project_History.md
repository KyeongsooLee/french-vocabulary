# TEF Vocab Loop — Full Project History & Product Decision Record

**Document purpose:** preserve the project’s memory so a future developer or a new GPT session can continue the app without losing small design intentions, user feedback, learning-engine philosophy, or reasons behind seemingly minor UI choices.

**Documented baseline:** `TEF_Vocab_Loop_v2_6.html`  
**Vocabulary corpus:** 2,866 unique study cards  
**Primary use case:** a Korean-speaking learner studying French vocabulary for practical use and TEF-oriented progression, especially through short, repeatable mobile study sessions.  
**Primary device/workflow:** Android phone and laptop; a downloaded single HTML file that can be opened directly is intentionally acceptable.

---

## 0. How to use this document

This file is deliberately more detailed than a normal changelog.

A changelog says **what changed**.  
This history also records:

- what the user noticed,
- why it felt wrong,
- what alternatives were considered,
- what product principle came out of the discussion,
- how the implementation changed,
- what must not be accidentally reverted later.

When a future change touches an existing feature, search this file for that feature before redesigning it.

A small-looking UI decision may carry a larger product intention. Example: the word-audio buttons and example card are intentionally visually separated even though both belong to answer feedback. That decision came from actual use and should not be “cleaned up” casually.

---

# 1. Original project goal

The project began as a personal French vocabulary app inspired by the useful parts of **말해보카-style** learning, but the goal was never to clone every feature.

The user wanted:

- a word-focused app rather than grammar/conversation/game clutter,
- short continuous questions,
- immediate feedback,
- multiple ways to know the same word,
- strong pronunciation support because French spelling does not transparently reveal pronunciation,
- delayed reappearance of mistakes,
- mobile-first use,
- offline/direct-open HTML if possible,
- persistent progress,
- backup/restore,
- coverage of the full uploaded vocabulary corpus.

A recurring design theme emerged early:

> **The app should optimize real retention, not merely make a large question bank.**

The user also cares about visual polish. Interfaces that technically work but feel bolted together, old-fashioned, overly dense, or internally inconsistent should be treated as product problems, not cosmetic trivia.

---

# 2. Vocabulary source and data preparation

Two vocabulary PDFs were used as the source corpus:

1. `LES VOCABULAIRES (A1-A2)-Coréen-MAJ.pdf`
2. `LES VOCABULAIRES-B1-B2(20260908-060108).pdf`

Extraction/normalization results established during development:

- A1-A2 source rows: 1,466 parsed vocabulary rows.
- B1-B2 source rows: 1,451 parsed vocabulary rows.
- Total source vocabulary rows: 2,917.
- Exact duplicates were collapsed **only within the same level**.
- A1-A2 unique study cards: 1,465.
- B1-B2 unique study cards: 1,401.
- Total unique study cards: **2,866**.
- 51 exact duplicate source rows were collapsed.
- No parsed vocabulary row with a Korean meaning was intentionally dropped.

### Important source-data rule

The Korean definitions from the PDFs are treated as source data and are preserved rather than silently rewritten to match outside knowledge.

This matters because a source definition may be awkward, overly broad, or occasionally linguistically imperfect. If an example sentence uses a more precise real-world sense, that does **not** automatically justify rewriting the stored Korean source definition.

Future developers should distinguish:

- **source definition** (`ko`)
- **example usage** (`exFr` / `exKo`)

rather than silently changing source definitions while “cleaning up” examples.

---

# 3. Early app phase: from flashcards to a loop-based learner

An earlier app existed as a broader flashcard-style system:

- `TEF_Vocab_Master.html`
- `TEF_Vocab_Master_Android_PWA.zip`
- related PWA files

It had:

- flashcards,
- binary rating,
- TTS,
- listening mode,
- IndexedDB,
- JSON backup.

The user confirmed that a downloaded single HTML file worked on Android simply by opening it.

This influenced a major technical choice:

> **Direct-open HTML is a valid supported delivery format.**

A PWA/home-screen install from local `file://` was not reliable because browser/service-worker installation rules expect a secure context such as HTTPS or localhost. An APK wrapper was discussed, but not built because the environment did not have the required Android SDK/Gradle toolchain and there was no reason to pretend a signed APK had been produced.

Native wrapping may be revisited later, but it is not currently a product requirement.

---

# 4. The 말해보카-inspired redesign

The user wanted a more active, continuous micro-quiz experience.

The resulting direction emphasized:

- short quiz loops,
- several modalities for the same word,
- pronunciation,
- delayed retry,
- fewer unnecessary settings/features,
- mobile speed,
- clear feedback,
- persistent progress.

The first loop-oriented build included:

- new-word introduction,
- French → Korean multiple choice,
- Korean → French multiple choice,
- audio → meaning,
- spelling input,
- errors reappearing after a few intervening questions,
- TTS,
- hard/starred words,
- IndexedDB,
- backup.

The user liked the direction, which led to a deeper redesign of the learning engine rather than merely adding more question types.

---

# 5. Core learning-engine philosophy

The largest conceptual change was the decision **not to represent a word with one single “strength” number**.

A learner can:

- recognize the written French word,
- understand it by ear,
- recall French from Korean,
- spell it accurately,

at very different levels.

Therefore each card tracks four independent skills:

- `meaning`: French → meaning recognition
- `listening`: audio → meaning
- `reverse`: Korean meaning → French
- `spelling`: typed French

### Why this matters

A listening mistake should not erase evidence that the learner knows the written meaning.

A spelling typo should not cause a word to be treated as globally unknown.

This remains a core invariant.

---

# 6. Word state model

The main visible lifecycle evolved into:

**New → Seen → Learning → Familiar → Mastered**

with **Weak** representing current instability that can interrupt a later stage and later recover.

Originally the system had:

- New
- Learning
- Familiar
- Mastered
- Weak

Later, real use revealed a missing state: **Seen**.

The distinction is extremely important and is documented in detail in Section 16.

---

# 7. “Seen is not learned” — original engine principle

Even before the dedicated Seen state existed, the engine philosophy was:

> **Exposure is not mastery.**

A new word should:

1. be introduced first,
2. then disappear for several intervening items,
3. then be tested,
4. then appear in another modality,
5. later be reviewed after real time has passed.

The app must never test a truly new word before showing it.

Repeated correct answers in one short session must not be treated as long-term memory.

---

# 8. Time-separated review design

The rough review ladder was designed around intervals like:

- same-session recheck,
- 1 day,
- 3 days,
- 7 days,
- 14 days,
- 30 days,
- 60 days.

The exact scheduling implementation may evolve, but the reasoning should remain:

> Same-session success is weaker evidence than success after time has passed.

Familiar and Mastered should therefore depend on spaced success rather than simple answer count.

---

# 9. Familiar, Mastered, Weak

### Familiar

Familiar represents a word that has demonstrated stable enough performance across the core skills, not merely one lucky recognition.

### Mastered

Mastered requires stronger, time-separated success. A word should not become Mastered purely because it was answered correctly several times in the same session.

### Weak

Weak was intentionally designed as a **current instability state**, not an eternal punishment for any historical error.

Important decisions:

- one mistake should not instantly mark a stable word Weak,
- repeated recent weakness in a specific modality may trigger Weak,
- Weak is skill-specific in cause,
- recovery requires successful spaced evidence,
- a single immediate retry should not fully “heal” a Weak state.

---

# 10. Error retry behavior

Errors should not repeat immediately.

Immediate repetition often measures short-term echo memory rather than actual retrieval.

The intended behavior:

- failed item returns after roughly 3–6 intervening questions,
- ideally it may return in another relevant modality later,
- only one pending retry per word/skill should exist at a time,
- retry logic must avoid infinite queue growth.

The engine therefore dynamically selects the next task rather than prebuilding one giant shuffled queue.

---

# 11. Dynamic task selection vs. blind shuffle

A key architecture decision was:

> Do not build the entire session as one shuffled list and hope the order works.

Instead, the engine chooses the next task dynamically using current state.

This allows the app to prioritize:

- weak items,
- due reviews,
- recently introduced words that need delayed testing,
- pending retries,
- new words,
- other study items.

It also supports automatic reduction of new-word load when review backlog grows.

---

# 12. Spelling: reinforcement, not a gatekeeper

Spelling is deliberately **not required for basic mastery**.

Reason:

The source contains many items that are awkward for strict single-string typing:

- `marier / se marier`
- `beau (bel) / belle`
- forms with parentheses,
- multiple lexical variants.

Therefore spelling is useful evidence, but should not prevent a learner from becoming Familiar/Mastered in the core vocabulary-learning sense.

The app uses spelling only for cards that fit a simpler answer shape.

Later QA on v2.5 found:

- 2,166 of 2,866 cards are spelling-eligible under the current `simpleFrench` rule.

Representative spelling behavior tested:

- `manger` → exact input accepted.
- `eleve` for `élève` → treated as accent-only / almost correct.
- early-stage omission of an article can be treated leniently in some cases.
- later-stage answer checking becomes stricter.
- clearly incorrect letter order remains wrong.
- complex forms such as `beau (bel) / belle` are excluded from spelling.

Spelling should remain a reinforcing modality rather than a source of unfair failure.

---

# 13. v2 architecture and QA mindset

The v2 engine introduced:

- independent skill states,
- spaced stages,
- delayed retry,
- retry deduplication,
- dynamic task choice,
- session retry limits,
- category-aware distractors,
- progress dashboard,
- IndexedDB persistence,
- migration support,
- session results,
- `dataVersion` handling.

A recurring development principle was established:

> **Do not trust the UI alone; simulate the engine.**

Useful QA scenarios include:

- always-correct learner,
- always-wrong learner,
- listening-only weak learner,
- Mastered → Weak → recovery,
- new word viewed but not answered,
- spelling accent error,
- duplicate retry prevention.

---

# 14. TTS metadata bug and pronunciation/display separation

A real-use issue appeared with cards such as:

`l'expression (f.)`

The app displayed source metadata correctly, but browser/Android TTS was being given the entire visible string, causing grammar metadata to be spoken.

The design correction:

- `fr` = original display string
- `tts` = pronunciation string

Examples:

- display: `l'expression (f.)`
- speak: `l'expression`

But the solution must **not blindly remove all parentheses**, because some parentheses represent real lexical variants rather than metadata.

Example:

- `beau (bel) / belle` contains real forms and should not be reduced as if `(bel)` were grammar metadata.
- `maillot (de bain)` may contain meaningful lexical content.

QA after the TTS cleanup reported:

- grammar metadata patterns remaining in TTS: 0
- stray terminal `f`/`m` caused by metadata stripping: 0

This separation between **display identity** and **speech text** should be preserved.

---

# 15. Example-sentence system

One French example and one Korean translation were added to every card.

Core UI decision:

> Examples are feedback/support, not answer clues.

Therefore examples are shown **after** the learner answers rather than before a quiz.

Each feedback section supports:

- normal sentence TTS,
- slow sentence TTS,
- word TTS,
- slow word TTS.

Examples were created using a mixture of:

- curated examples,
- targeted manual/reviewed overrides,
- controlled templates.

It would be inaccurate to claim that every one of the 2,866 examples was individually publication-edited by a human.

### Example-quality improvement pass

A later quality pass corrected weak generic/adjective examples, including cases like:

- bad generic form: `C'est heureux.`
- improved: `Il est heureux dans son nouveau travail.`

Other human-state adjectives and common constructions received more natural usage examples.

A previous generic “safe placeholder” category was eliminated from the final example set at that time.

QA confirmed:

- missing French examples: 0
- missing Korean examples: 0
- safe placeholder examples: 0

However, many template examples remain intentionally simple. Their role is to show word usage, not necessarily to become high-value sentence-learning material.

This distinction later motivated the dedicated Sentence tab.

---

# 16. Critical real-use feedback: introduction-only words were incorrectly counted as Learning

This is one of the most important product decisions in the project.

### User observation

The user entered study, viewed one new word, pressed `확인했어요`, then exited.

The dashboard counted that word as `Learning`.

The user questioned whether that was conceptually correct.

### Problem diagnosis

At the time, pressing `확인했어요` set `introduced=true`, and any introduced word with no higher skill progress fell into `Learning`.

That meant:

**seen once = learning**

which contradicted the earlier learning philosophy.

### Product decision

A new visible state, **Seen**, was created.

Meaning:

- **New**: never shown
- **Seen**: introduction viewed, but no actual quiz answer yet
- **Learning**: at least one real learning response has started
- **Familiar**
- **Mastered**
- **Weak**

### Core phrase

> **봤다 ≠ 배웠다**  
> Seeing is not learning.

### Resume behavior

Seen words should not necessarily replay the entire introduction every time.

On the next study session, they can resume from the first real meaning test.

### Priority decision

Seen words are unfinished learning and should be surfaced with high priority.

The resulting direction became approximately:

**Weak → Seen → due review → New → other**

This prevents “half-started” words from disappearing indefinitely.

### Backward compatibility

Existing words that had been introduced but had zero real core attempts could be classified as Seen without requiring the user to reset progress.

---

# 17. Whole-topic custom study exposed source-order bias

### User observation

The user selected a level, chose **전체 주제**, and started study.

It felt like the app was still serving verbs first in source order.

### Diagnosis

The candidate pool was prioritized correctly at a high level, but equal-priority new words were ultimately selected according to card/source ID order before shuffling within the chosen batch.

Because the source PDF began with verbs, the first selected batch could be almost entirely verbs.

So the effective behavior was closer to:

1. sort the entire pool,
2. take the first 20,
3. shuffle those 20.

Rather than:

1. draw a balanced 20 from the whole eligible pool.

### Product decision

For **전체 주제**, candidate selection should be **category-balanced**, not merely random and not source-order biased.

Important nuance:

- review/weak priorities are preserved,
- the balancing happens inside the relevant priority bucket,
- if the user explicitly selects one category, that category is respected,
- the goal is diversity, not mathematically equal quotas.

A QA sample after the change produced 20 A1-A2 new words spanning roughly 20 different categories rather than a verb-heavy block.

---

# 18. New-word accidental tap and the “back” requirement

### User observation

On a new-word introduction screen, the learner may accidentally press `확인했어요` and move on.

### Desired behavior

The user wanted a way to go back.

### Important implementation decision

A full state rollback was considered riskier than necessary.

Instead, the app added a **previous-introduction replay**:

- a back arrow becomes available,
- it shows the previously introduced word again,
- the learner can hear the word again,
- returning from that replay resumes the current task,
- it does **not** mutate/reverse progress or reschedule the engine.

This is intentionally a **safe replay**, not a transactional undo.

Reason:

The UX problem is “I accidentally moved past the card,” not “rewind the entire learning engine.”

Future developers should preserve that distinction unless a true undo system is deliberately designed.

---

# 19. Answer-feedback micro-UI: word audio vs. example card

This is a small visual decision with explicit user feedback behind it.

### Initial state

The answer feedback showed:

- answer result,
- word/meaning/status,
- example card,
- word audio controls separately below.

The user marked up a screenshot and indicated:

- the extra system-like explanatory sentence under the status was unnecessary,
- the word-audio controls felt detached from the feedback structure.

### First interpretation/change

The redundant explanatory line was removed.

The word-audio buttons were moved into the example area so the feedback would feel more organized.

### User correction

After seeing the modified version, the user said the word-audio controls and example now looked **merged**, specifically because they shared the same white background/card.

This was not the intended visual relationship.

### Final design decision

The final structure should be:

- green/answer feedback region
  - answer result
  - word + meaning
  - skill/status line
  - **word audio controls**
- separate white example card
  - example label
  - French example
  - Korean translation
  - example audio controls
- Next button

### Why

The word audio and example belong to the same overall feedback phase, but they serve different semantic roles.

They should be close, but not visually collapsed into one component.

This is an example of a “minor” UI choice that future cleanup must not accidentally reverse.

---

# 20. Word tab becomes a library, not just a list

### User idea

The user wanted to tap a word in the Word tab and see more information “like a library.”

### Product direction

The Word tab should function as a personal vocabulary library/dictionary, not merely as a searchable list.

### v2.4 detail panel

A mobile-friendly bottom sheet was preferred over a full navigation jump.

Reason:

- preserves list position,
- feels natural on mobile,
- easy to dismiss,
- supports browsing many words quickly.

The detail sheet includes:

- French word,
- Korean meaning,
- level,
- category,
- current state,
- word TTS,
- slow word TTS,
- example,
- Korean example translation,
- example TTS,
- slow example TTS,
- skill scores for meaning/listening/reverse/spelling,
- last-study information,
- next-review information,
- correct/wrong stats,
- hard/star toggle.

### Explicit user request

PDF source/page information was **not necessary** in the library view and was removed from that UI.

The underlying source data may still exist internally; the user simply does not need it displayed in this context.

---

# 21. Session continuity: interrupted study should be resumable

### Product motivation

On a phone, interruptions are normal:

- app switching,
- accidental close,
- phone calls,
- leaving the session intentionally.

A mobile learning app should not treat these as catastrophic.

### v2.5 direction

A live study session is persisted so the home screen can show:

**진행 중인 학습**

with:

- study source/type,
- completed count / total,
- progress percentage,
- last-active information,
- `이어하기`,
- `새로 시작`.

### Important behavior

The saved session includes enough structure to preserve:

- remaining target words,
- pending retry tasks,
- task availability order,
- original mode,
- progress counts.

### Exit behavior

The close/quit action saves the session before leaving.

### Completion behavior

Once a session fully completes, the saved active session is cleared.

### Study-time nuance

Elapsed wall-clock time across a long interruption should not count as actual study time.

The implementation therefore tracks active time more defensibly rather than simply subtracting start timestamp from completion timestamp.

### Preservation principle

If a future developer changes session serialization, old progress data and in-flight-session compatibility must be considered explicitly.

---

# 22. Spelling QA performed during v2.5

The user specifically asked whether the spelling feature actually worked, since they had not personally tried it yet.

Static/logic QA was performed.

Representative passing checks:

- simple French word eligibility,
- complex multi-form exclusion,
- exact spelling,
- accent-only tolerance,
- early article tolerance,
- later stricter article requirement,
- obvious wrong spelling,
- graded hint generation.

Result at the time:

- 2,166 / 2,866 cards eligible for spelling under the current rule.

This feature should be tested on a real mobile keyboard as well whenever possible, because static JS tests cannot fully reproduce Android keyboard/IME behavior.

---

# 23. Sentence tab concept: examples as a second kind of library

### User idea

The user proposed another tab where examples could be browsed directly.

They questioned the usefulness of trivial sentences such as:

> “나는 시금치를 좋아한다.”

The important insight was:

> Not every acceptable word example deserves to become a sentence-learning item.

### Product separation

The roles became:

- **Word tab** = vocabulary dictionary/library
- **Sentence tab** = reusable expression/pattern reading space

### Recommendation philosophy

Recommended sentences should have learning value beyond merely proving that a noun can appear in a sentence.

High-value features include:

- verb + preposition patterns,
- common collocations,
- reusable B1-B2 structures,
- conjunction patterns,
- constructions useful in TEF writing/speaking,
- sentences that reveal actual usage constraints,
- useful modal/argumentative phrasing,
- natural, reasonably short sentences.

Examples of the intended value:

- `s'adapter à`
- `dépendre de`
- `permettre de`
- `prendre une décision`
- `réduire les coûts`
- `atteindre un objectif`
- `Il est important de…`
- `Cette mesure vise à…`
- `Même si…`

### What should usually not be recommended

Simple noun-confirmation sentences can remain in the full example library but need not appear in Recommended.

Examples conceptually like:

- “I like spinach.”
- “The cat is small.”

may be perfectly acceptable vocabulary examples but provide limited reusable sentence structure.

### Important nuance

Recommendation is **not based only on level**.

A simple A1/A2 structure such as `avoir besoin de` may be highly reusable and deserve recommendation.

---

# 24. v2.6 Sentence Library implementation

The bottom navigation became:

**오늘 · 학습 · 단어 · 문장 · 설정**

The Sentence tab supports:

- Recommended / All toggle,
- sentence search,
- level filter,
- category filter,
- normal sentence audio,
- slow sentence audio,
- link back to the associated word detail.

### Recommendation scoring

The current implementation uses a heuristic score, rewarding things such as:

- curated/reviewed sentence quality,
- useful grammatical/construction patterns,
- verb-preposition structures,
- reusable discourse/connective patterns,
- moderate sentence length,
- B1-B2 utility,
- modal/argumentative structures.

It penalizes generic template patterns such as:

- `On parle souvent de ... dans les médias.`
- `On utilise souvent ... dans cette situation.`
- other obviously generic placeholder-style wording.

### Current recommendation set

At v2.6 QA:

- total examples/cards: 2,866
- recommended: **353**
- A1-A2 recommended: 78
- B1-B2 recommended: 275
- source quality among recommended:
  - curated: 269
  - reviewed: 84

### Interpretation

This does **not** mean only 353 examples are “good.”

It means 353 passed a deliberately stricter threshold for:

> “worth browsing/repeating as a sentence-learning item independent of the word card.”

The other examples may still be completely adequate as word-usage examples.

This distinction should be preserved in future UI copy and documentation.

---

# 25. Product tone and UI preferences learned from feedback

The user repeatedly prefers:

- modern, uncluttered UI,
- obvious grouping,
- short practical labels,
- visible separation between semantically different components,
- mobile-first behavior,
- not overloading the app with features that do not improve learning.

The user dislikes:

- system-like explanatory chatter inside study feedback,
- components that look “stuck on” or bolted together,
- excessive visual merging of unrelated controls,
- source-order behavior that feels non-random/unbalanced,
- status labels that overclaim learning,
- needless complexity when a small reliable solution works.

A useful product test is:

> “Would this feel like a coherent learning app on a phone, or like a developer demo with features added one by one?”

---

# 26. Current major modules in v2.6

## Today/Home

Shows:

- today’s learning availability,
- review/new-word context,
- progress,
- streak,
- state counts,
- active-session resume card when applicable.

State counts include:

- New
- Seen
- Learning
- Familiar
- Mastered
- Weak

## Study

Supports:

- level selection,
- category selection,
- session size,
- focus mode,
- whole-topic category-balanced selection,
- new-word introduction,
- delayed testing,
- retries,
- skill-specific progress,
- previous-introduction replay,
- saved/resumable session.

## Word

Supports:

- search,
- level/category filter,
- word list,
- pronunciation,
- star/hard marker,
- detailed bottom-sheet library view.

## Sentence

Supports:

- Recommended vs All,
- search,
- level/category filters,
- example TTS,
- slow TTS,
- linked word detail.

## Settings

Supports:

- daily new-word target,
- review limit,
- voice,
- speech rate,
- backup/export,
- restore/import,
- reset.

---

# 27. Persistence model

The app uses browser-side persistence (IndexedDB with fallback logic).

Important distinction:

- the HTML contains the **app and static vocabulary data**,
- the learner’s actual progress is stored separately in browser storage,
- exporting the JSON backup is necessary if the learner wants to move/analyze/restore that personal progress elsewhere.

Therefore:

> HTML alone is enough to continue **development**.  
> HTML + exported JSON is needed to fully carry over the learner’s **actual progress state**.

---

# 28. Versioning philosophy

The project has used incremental single-file versions.

Relevant milestones include:

- `TEF_Vocab_Master.html` — earlier broader flashcard app
- `TEF_Vocab_Loop.html` — first loop-oriented version
- `TEF_Vocab_Loop_v2.html` — stronger learning engine
- v2.1 — examples/TTS quality work and feedback layout iteration
- v2.2 — category-balanced study + previous-introduction replay
- v2.3 — Seen state
- v2.4 — word-library detail sheet
- v2.5 — resumable sessions + spelling QA
- v2.6 — sentence library + recommended-example scoring

Future releases should keep a previous stable backup when making nontrivial changes.

---

# 29. QA limitations

The development environment has not reliably supported full browser GUI automation for the local HTML.

Previous attempts with Chromium/Playwright encountered environment restrictions such as local/file URL blocking and browser environment issues.

Therefore QA has primarily relied on:

- static code inspection,
- JavaScript syntax checking,
- embedded-data integrity checks,
- targeted Node logic tests,
- learning-engine simulations,
- card-count assertions,
- TTS metadata checks.

Final UI behavior still benefits from real Android testing by the user.

This limitation should be stated rather than pretending full end-to-end GUI automation has occurred.

---

# 30. Data integrity invariants

Unless intentionally changed with a migration plan:

- unique vocabulary card count should remain 2,866,
- card IDs should remain stable,
- progress records should continue to map to the same IDs,
- example fields should not disappear,
- `tts` should continue to be separate from display `fr`,
- source Korean meanings should not be silently rewritten,
- `dataVersion` / migration behavior should be respected.

---

# 31. Core learning invariants — compact form

These are the most important rules a future developer should protect:

1. New words are introduced before testing.
2. Seeing an introduction is not Learning; it is Seen.
3. A real answer starts Learning.
4. Meaning/listening/reverse/spelling are independent.
5. Spelling is not required for mastery.
6. One modality error does not erase other skill progress.
7. One mistake does not instantly make a stable word Weak.
8. Weak is recoverable.
9. Same-session success is weaker than spaced success.
10. Errors reappear after intervening questions.
11. Retry tasks are deduplicated.
12. Whole-topic study uses category-balanced selection.
13. Previously Seen unfinished items are prioritized for continuation.
14. Previous-introduction “back” is a safe replay, not engine rollback.
15. Active sessions are resumable.
16. Direct-open Android HTML remains a supported workflow.

---

# 32. Micro-decisions worth remembering

These are small decisions that can easily be lost:

- The redundant “system explanation” sentence beneath answer status was intentionally removed.
- Word audio controls should not share the white example-card background.
- Example audio belongs inside the white example card.
- Word audio belongs in the answer-feedback area.
- PDF page/source is not shown in the Word library UI.
- The Word detail is a bottom sheet rather than a full page to preserve browsing context.
- The Sentence tab defaults to Recommended, not All.
- “Recommended” means sentence-learning value, not simply grammatical correctness.
- A simple example can be useful for a word card yet intentionally absent from Recommended.
- Whole-topic study should feel mixed across topics instead of reflecting PDF order.
- The new-word previous button is there because accidental taps happen in real phone use.
- Session time should not count hours/days when the app is closed.
- Progress/state names must not overstate learning.

---

# 33. Known future work discussed

Not all items below are committed. They are directions discussed as useful next steps:

### Higher priority

- Improve distractor quality:
  - same category,
  - same part-of-speech where possible,
  - semantically confusable choices,
  - avoid absurdly easy distractors.
- Improve Word-library filters:
  - New / Seen / Learning / Familiar / Mastered / Weak / Starred.
- Continue reviewing sentence/example quality, especially generic B1-B2 template-heavy areas.

### Later / optional

- Context/cloze questions using strong example sentences.
- More sophisticated recommendation scoring for Sentence tab.
- Manual favorite/star system for sentences.
- Native Android wrapper if direct HTML eventually becomes limiting.
- Background audio only if moved to a native/media architecture.

### Important caution

Do not add features simply because they are possible. The app’s value comes from fast, focused vocabulary study.

---

# 34. Recommended workflow for all future changes

For every meaningful update:

1. Read `PROJECT_HANDOFF.md`.
2. Search this history for the affected feature.
3. Inspect the current HTML implementation.
4. Make the smallest change that satisfies the product intent.
5. Preserve card IDs/data and progress compatibility.
6. Run JS syntax QA.
7. Run relevant targeted logic/data checks.
8. Record:
   - user feedback,
   - reason,
   - implementation,
   - QA,
   - unresolved concerns.
9. Update `CHANGELOG.md`.
10. Update `PROJECT_HANDOFF.md` if the current architecture changed.

This documentation work is part of the feature, not an optional afterthought.

---

# 35. Documentation philosophy established by the user

The user explicitly wants future sessions to retain:

- major architecture,
- tiny UI preferences,
- reasons behind modifications,
- rejected/adjusted interpretations,
- behavioral nuance.

Therefore future documentation should avoid summaries that say only:

> “Moved button.”

Prefer:

> “Moved button because the user felt it was visually detached; first attempt over-grouped it with the example, which the user then corrected; final layout keeps the controls in the same feedback phase but visually separates their backgrounds.”

The goal is to preserve **intent**, not merely implementation.

---

# 36. Current baseline

At the time this history was created, the baseline application is:

`TEF_Vocab_Loop_v2_6.html`

Key status:

- 2,866 unique cards
- full example coverage
- skill-specific learning engine
- Seen state
- category-balanced whole-topic study
- safe previous-introduction replay
- resumable sessions
- spelling implementation tested statically
- word library detail sheet
- sentence library with 353 recommended sentence-learning examples
- IndexedDB persistence and JSON backup
- direct-open HTML works for the user’s Android workflow

This file should be read together with `PROJECT_HANDOFF.md`, `CHANGELOG.md`, `QA_NOTES.md`, and `NEXT_SESSION.md`.
