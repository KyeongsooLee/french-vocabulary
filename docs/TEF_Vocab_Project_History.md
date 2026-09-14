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

# 37. v2.7 — Today becomes Smart Session

The user clarified that daily study volume is inherently variable:

- some days may contain only ~10 interactions,
- other days may contain ~100 or more.

This made fixed daily New/review quotas a poor fit for the real usage pattern.

The product decision became:

> The user decides how long to study.  
> The engine decides what the next useful task should be.

## What changed

Today Study was converted from a prebuilt finite list to a continuous dynamic selector.

The engine now repeatedly considers:

- Weak work,
- overdue review,
- Seen unfinished learning,
- delayed retry,
- New introduction,
- reinforcement only when necessary.

The old visible settings for daily New count and maximum review count were replaced with:

- 적게
- 보통
- 많이

for New-word pace.

## Cross-mode concern discovered before implementation

The user asked whether studying in both Today and Custom could corrupt progress.

This led to an important architecture rule:

> Today and Custom are different session experiences operating on one shared learner model.

v2.7 therefore separated temporary active-session state while preserving one global card record.

A Today retry can become stale if Custom later studies the same skill. Saved tasks are revalidated rather than blindly replayed.

## Migration decision

The old v2.6 Today session queue could not safely be carried over because it represented a frozen future list, which contradicts the new dynamic scheduler.

Therefore:
- already-earned card progress is preserved,
- old Today future queue is retired,
- finite Custom session can be retained.

## Pre-release problem found by simulation

When the first Smart implementation was simulated, Weak cards dominated long sessions because the global Weak state may remain until recovery on a later day.

The fix did NOT change the global Weak algorithm in v2.7.

Instead, Smart Session received a session-local anti-spam cooldown:
- a Weak card recently served is temporarily excluded from repeated global Weak selection,
- explicit retries after a wrong answer remain allowed.

This preserves the planned release split:
- v2.7 = Smart/session synchronization
- later engine-hardening release = Weak/Mastered redesign

## Delay correction

Testing also showed that the old turn arithmetic could produce only two actual intervening interactions even when the code said `+3`.

Smart Session delay math was corrected so the first test of a newly introduced word has at least three real intervening interactions.

This is an example of why implementation was simulated before release.
---

# 37. Real-device feedback after v2.7 — Auto mode omitted spelling

After testing v2.7 on the phone, the user reported that everything felt good except one concrete issue:

> When the focus method was automatic, spelling/typing questions never appeared.

Inspection confirmed this was not bad luck. It was structurally impossible in the automatic path.

The app intentionally defined `CORE` as:
- meaning
- listening
- reverse

because spelling is reinforcement and must not gate mastery.

However, the same `CORE` list was also being used as the source of automatic question selection. This accidentally turned the product principle “spelling is not required for mastery” into the unintended behavior “spelling is never selected automatically.”

Decision:
- keep spelling outside mastery CORE,
- but include it in automatic practice as a secondary reinforcement skill.

v2.7.1 behavior:
- simple spelling-eligible words can receive typing questions automatically,
- new-word secondary practice uses spelling at a modest rate,
- mature auto reviews can also surface spelling at a lower rate,
- Weak/core due reviews remain more important,
- forced 철자 mode continues to behave as before.

Product principle clarified:

> **A skill can be optional for mastery while still being part of normal automatic practice.**

---

# 38. Real-device feedback — number answers should be numerals

The user noticed that French number questions displayed Korean number words as meanings/answers, for example:

- `vingt` → `스물`
- `quatre-vingts` → `여든`

This felt unnatural for number learning. The user preferred the actual numeric representation.

Decision in v2.7.2:

- preserve the original PDF Korean meaning in static source data,
- add a presentation layer for true numeric vocabulary,
- display Arabic numerals in learning UI.

Examples:

- `un` → `1`
- `dix-sept` → `17`
- `trente et un` → `31`
- `cent un` → `101`
- `mille` → `1,000`
- `million` → `1,000,000`

Important nuance:
- only actual number-value cards are converted,
- category neighbors such as `le nombre`, `le numéro`, `le chiffre`, `le moment` keep their Korean meanings,
- number distractors are restricted to other numeric cards so answer choices remain coherent,
- source `ko` values are not rewritten.

Product principle:

> For numerical vocabulary, the learner should connect French number words directly to the number itself rather than through an extra Korean-number-word layer.

---

# 39. Number-display refinement — Arabic below 100, Korean units from 100

After v2.7.2 converted all number-value cards to Arabic numerals, the user refined the preference.

Small values are easiest to recognize directly as numbers:
- 1
- 21
- 80
- 92

But from 100 upward, Korean large-number words are more useful and natural for the learner:
- 백
- 천
- 만
- 백만
- 억
- 백억

Decision in v2.7.3:
- 1–99: display Arabic numerals.
- 100 and above: display the original Korean source meaning.

The source vocabulary data remains unchanged.

Product reason:
> Small numbers benefit from immediate numeric recognition, while larger values benefit from Korean unit grouping because `백/천/만/억` is cognitively clearer than long strings of digits.

---

# 40. Bilingual Korean / English study mode

The user wanted to preserve the Korean app while also studying French through English.

The project therefore did **not** create a separate English app or a second progress database. v2.8.0 keeps one French corpus and one learner model, then changes the presentation/study language.

## Data structure

The original source fields remain intact:
- French word (`fr`)
- Korean meaning (`ko`)
- French example (`exFr`)
- Korean example (`exKo`)

English content is stored as a sidecar keyed by the same stable card ID:
- English meaning
- English example translation

This protects the PDF-derived Korean source data from being overwritten.

## Shared-progress decision

Korean and English modes share:
- New / Seen / Learning / Familiar / Mastered / Weak
- meaning/listening/reverse/spelling records
- review timing
- stars
- sessions

The switch is treated as another way to study the same French vocabulary, not a separate course.

## Language switch behavior

Settings now offers:
- 한국어
- English

The selection updates:
- interface labels
- meanings
- example translations
- category display labels
- quiz prompts/choices
- Word library
- Sentence library

If the learner switches languages while an unanswered quiz is open, the current question is rerendered so old-language choices cannot be marked against a new-language expected answer.

## Number behavior

The existing preference is preserved:
- 1–99: Arabic numerals in both modes.
- 100+: Korean mode uses `백/천/만/억...`.
- 100+: English mode uses `one hundred / one thousand / one million...`.

The full set of 58 numeric cards is now separate from the 1–99 display override, fixing a subtle v2.7.3 issue where 100+ cards were not treated as numeric cards for distractor filtering.

## English-content quality

All 2,866 English meanings and 2,866 English example translations are present and structurally validated.

They were generated in bulk and were not individually human-edited one by one. Actual study feedback should continue to identify awkward wording, ambiguous meanings, or translation nuance.

---

# 41. Bilingual UI regression — bottom navigation labels became too large

After v2.8.0, the user compared the new app with the previous version and noticed that the bottom navigation labels (`오늘 / 학습 / 단어 / 문장 / 설정`) looked noticeably larger.

The bilingual update had changed the nav markup from:

`<span>icon</span>오늘`

to:

`<span>icon</span><span data-i18n="...">오늘</span>`

The existing CSS rule `.bottom span { font-size:20px; ... }` was originally intended only for the icon span. Once the label itself became a span for localization, it accidentally inherited the icon's 20px size.

v2.8.1 fixes the selector rather than changing the overall nav design:

- first child icon span: 20px
- translated label span: inherits the original 10px button font size

Product intent:
> Bilingual localization must not alter the compact visual scale of the original bottom navigation.

---

# 42. Spelling UI simplification and TTS grammar-marker cleanup

The user tested spelling on the phone and preferred ordinary continuous typing over the visual `_ _ _` style letter slots.

Decision in v2.8.2:

- show the meaning/prompt,
- show a normal French text input,
- do not reveal a spelling pattern by default,
- offer a small optional `힌트 보기 / Show hint` button,
- reveal only the beginning when requested.

Examples:
- `brosse à cheveux` → `b…`
- `l'expression` → `l'e…`
- `le travail` → `le t…`

The hint is intentionally small. It should help retrieval without turning the task into a fill-the-blanks puzzle.

## Hint scoring

The user had already agreed with the principle that a hinted correct answer should still be accepted but should be weaker learning evidence.

Implementation:
- hinted answer counts as correct,
- but does not advance the spelling step,
- spelling remains non-mandatory for mastery.

## TTS grammar-marker bug

The user also heard TTS pronounce grammar notation such as `+ inf`.

A scan found 20 TTS strings still containing shorthand such as:
- `+ inf`
- `+ ind`
- `+ sub`
- `+ cond`
- `qn`
- `qc`

Those 20 TTS strings were replaced with speakable lexical forms, and `cleanSpeech()` now has a fallback that strips such notation if it appears again.

Important distinction:
- visible vocabulary notation can remain useful for learning,
- pronunciation audio should speak the lexical expression, not grammar abbreviations.

---

# 43. Streak calendar visualization

The user wanted the `연속 N일` streak number to be visually meaningful rather than only a number.

Decision in v2.8.3:
- tapping the streak pill opens a calendar,
- studied dates are filled in purple,
- today is visually outlined,
- previous/next months can be browsed,
- the visible month shows how many days were studied,
- the current streak remains visible in the calendar summary.

The calendar uses the existing `studyDays` object. No second attendance/history system was created.

Product intent:
> The streak should be something the learner can *see* as a pattern of consistency, not just read as a counter.

---

# 44. Responsive calendar rebuild after Android layout failure

The first streak-calendar implementation looked acceptable structurally but failed on the user's Android device.

Observed real-device problems:
- studied-day circles became much too large,
- date columns no longer visually matched the weekday headings,
- dates overflowed toward the right edge,
- the final week collided with the legend/close area,
- the modal became unnecessarily tall.

Root cause:
The calendar used `aspect-ratio: 1/1` on each entire seven-column grid cell. On some mobile/browser viewport calculations, the grid cell itself became the date circle, so any width expansion directly enlarged every marker and row.

v2.8.4 redesign:
- the grid cell now handles only layout/alignment,
- a nested `calendarDate` element handles the small visual circle,
- seven columns use `repeat(7, minmax(0,1fr))`,
- date-marker size is capped with `clamp()`,
- modal width uses the viewport with a desktop maximum,
- modal height uses dynamic viewport units and internal overflow,
- extra media rules protect very narrow and very short screens.

Product principle:
> Responsive design should constrain components by role, rather than letting a layout cell also determine the visual size of its contents.

---

# 45. Today-date visual state in the calendar

The user refined the calendar behavior after the responsive rebuild.

Desired behavior:

- **Today, before studying**
  - date number has a distinct color,
  - circle is hollow/unfilled.

- **Today, after studying**
  - circle fills purple exactly like another studied day,
  - the date number keeps the distinct today color.

This creates two independent visual signals:

1. fill = whether the day was studied,
2. date-number color = whether the date is today.

v2.8.5 implements this separation.

---

# 46. Calendar today-marker vertical offset caused by CSS class collision

Real Android testing showed that only today's date circle was vertically lower than the other dates.

Root cause was identified precisely:

- the Home hero already used the generic CSS class `.today`,
- the calendar also added `today` to today's date cell,
- therefore the calendar cell inherited Home-layout CSS such as `display:grid`, `margin-top:20px`, and responsive `.today` rules.

This moved only today's calendar cell downward.

v2.8.6 renames the calendar state class to `isToday`, isolating it from the Home layout.

Product/engineering lesson:
> Avoid generic state class names that can collide with existing component classes; calendar state classes should be component-scoped.

---

# 47. v2.9.0 Example Quality Upgrade — first large pass

The user wanted to address a major content problem before further engine work: many example sentences technically contained the target word but offered almost no useful learning value, such as `J'aime ...` or bare `C'est ...` examples.

The project decision is now:

> An example should teach how the word is actually used, not merely prove that the word can appear in a sentence.

v2.9.0 begins this content-quality pass and changes **286** examples.

Priority areas in this first pass:
- food and kitchen vocabulary,
- clothing and accessories,
- sports,
- drinks,
- A1-A2 adjective usage,
- malformed/weak A1-A2 `ADVERBES` and `OBJETS` examples.

Each changed card updates three aligned pieces together:
- French example,
- Korean translation,
- English translation.

The source vocabulary meaning itself is not changed.

Pedagogical criteria used:
- show a realistic situation,
- teach a useful verb/preposition/collocation where possible,
- keep the sentence short enough to study,
- avoid artificial `I like X` noun demonstrations,
- prefer sentences reusable in daily life or TEF-style expression,
- preserve natural French rather than forcing the dictionary display form mechanically.

The change list is stored in `EXAMPLE_UPGRADE_v2_9.tsv`.

This is intentionally a staged project. B1-B2 adjective placeholders and several other repetitive template families remain for later passes rather than being blindly regenerated all at once.

---

# 48. v2.9.1 Example Quality Upgrade — pass 2

The user asked to continue immediately after the first 286-example rewrite.

This pass changes another **167** examples.

Three high-noise template families were targeted:

1. 80 B1-B2 adjective cards using bare `C'est [adjectif].`
2. 34 society cards using `On parle souvent de X dans les médias.`
3. 53 location cards using `Nous passons près de X.`

The goal was not merely variety. Each replacement tries to reveal a useful construction, collocation, or realistic use:

- `fier de`
- `curieux de savoir`
- `impropre à la consommation`
- `populaire auprès de`
- `opposé à`
- `demander l'asile`
- `accord de paix`
- `à mon avis`
- `se trouver devant / près de`
- `aller à la poste pour...`
- `prendre l'autoroute`
- `rester dans la voie de droite`

French, Korean, and English example text were kept aligned.

The original vocabulary meanings and stable card IDs remain untouched.

---

# 49. v2.9.2 Example Quality Upgrade — pass 3

After the first two content passes, the next target was not merely repetitive wording but examples that were actively poor learning material.

Four A1-A2 template families were especially weak:

- `Nous passons près du ...`
- `Je parle souvent avec ...`
- `Le médecin examine ...`
- `On voit souvent ... dans la nature`

Some were grammatically possible but taught almost nothing; others were unnatural for words such as `le peuple`, `le genre humain`, or family relations.

v2.9.2 rewrites **159** examples.

The new examples deliberately teach usable situations and collocations:
- place + actual function or errand,
- family relation + plausible life context,
- body part + common symptom/injury/action,
- nature noun + characteristic verb or physical context.

The example-quality project remains staged. The next obvious bulk families are in B1-B2 environment, news, religion, and history.

