# TEF Vocab Loop v2.7 — QA Report

**Release:** `TEF_Vocab_Loop_v2_7.html`  
**Focus:** Today Smart Session + Today/Custom shared-progress synchronization  
**Important:** Weak/Mastered redesign was intentionally NOT included in this release.

---

# 1. What changed

## Today Study

Today Study no longer builds a fixed daily list such as:

- new 20
- review 40

Instead it is now a continuous **Smart Session**.

The learner can stop after any amount:

- 10 interactions
- 30 interactions
- 100+ interactions

After every interaction, the engine reevaluates what should come next.

Candidate types include:

- current Weak work
- overdue Due review
- Seen first test
- delayed retry
- New introduction
- light reinforcement when there is genuinely nothing else due

---

# 2. New-word pacing

The old settings:

- 하루 새 단어
- 최대 복습

were removed from the visible v2.7 Settings UI.

They were replaced with:

**새 단어 속도**
- 적게
- 보통
- 많이

The setting affects how quickly Smart Session earns eligibility to introduce a new word.

The engine automatically slows New introductions when:

- Due backlog is high
- Seen backlog is high
- Weak backlog is high

With a very large review debt, New words may temporarily disappear entirely. This is intentional: the engine is allowed to prioritize unfinished/review work rather than continuously adding new exposure.

---

# 3. Today and Custom can coexist

v2.6 stored only one active session.

v2.7 separates session state into:

- `activeSessions.today`
- `activeSessions.custom`

Therefore a user can:

1. study in Today,
2. pause it,
3. do a Custom session,
4. return to Today,

without the Custom session deleting the Today session.

Both modes still update the same global word-progress record.

---

# 4. Shared-progress synchronization

Card learning state remains global.

Example:

1. Today creates a retry for `atteindre`.
2. User leaves Today.
3. Custom Study successfully answers the same skill.
4. User returns to Today.

The old Today retry is revalidated.

If the global skill has been attempted after that retry was created, the stale retry is discarded rather than replayed blindly.

The same idea is applied to finite Custom session tasks where possible.

---

# 5. Seen synchronization

Seen remains global.

Tested behavior:

- card introduced in Custom is no longer New in Today,
- card introduced in Today is no longer New in Custom,
- an untouched New intro saved in an old finite session is revalidated if another mode already studied it.

This preserves the product rule:

> Seeing is not Learning, but seeing is still a global event.

---

# 6. Delay behavior

For Today Smart Session:

- New introduction → first meaning test after **3–5 other interactions**
- Wrong answer → retry after **3–5 other interactions**
- successful first meaning test → secondary listening/reverse test after **3–6 other interactions**

A runtime gap test over repeated Smart runs observed a minimum of **3 actual intervening interactions** before a first New-word test.

This is stronger than the older turn-gap implementation.

---

# 7. Weak anti-spam safeguard

The global v2.6 Weak model is intentionally unchanged.

Because a Weak card can remain Weak until spaced recovery on another day, a naive Smart selector repeatedly chose the same Weak cards.

A session-local safeguard was therefore added:

- after a Weak card is served through the global Weak selector,
- it is temporarily excluded from that selector for roughly 40 Smart turns,
- explicit delayed Retry tasks can still appear earlier after a wrong answer.

This prevents current Weak cards from monopolizing long Smart Sessions without changing the underlying Weak learning-state rules.

The full multi-skill Weak redesign is still deferred to the next engine-hardening release.

---

# 8. Day-to-day usage

A Smart Session can remain resumable, but its visible daily counters reset when the saved Smart session crosses into a new local calendar day.

The scheduler state can continue, while visible values such as:

- interactions
- correct/wrong
- new count
- review count
- active time

restart for the new day.

This supports the user’s real pattern:

- one day 10 interactions,
- another day 100 interactions,

without creating a required daily quota.

---

# 9. v2.6 migration behavior

## Card progress

Preserved.

Existing:

- Seen
- Learning
- Familiar
- Mastered
- skill progress
- stars
- due dates

remain in the global progress data.

## Old v2.6 Today active session

Not carried forward as a fixed future queue.

Reason:

v2.6 Today session was based on a frozen target list, while v2.7 Today is dynamically selected after every interaction.

The obsolete future Today queue is retired, but all card-level learning already earned remains.

## Old v2.6 Custom active session

Retained where structurally valid because Custom remains a finite target-list session.

---

# 10. Storage/schema change

v2.7 introduces:

- `APP_VERSION = 2.7.0`
- `DATA_SCHEMA_VERSION = 3`
- `SESSION_SCHEMA_VERSION = 2`

v2.7 uses new storage keys:

- IndexedDB: `loopStateV3`
- localStorage fallback: `tef_vocab_loop_v3`

It still reads v2 storage as a migration source.

This deliberately avoids writing the new schema over the old v2 state.

---

# 11. Backup behavior

v2.7 exports:

`tef-vocab-loop-v2_7-backup.json`

Import supports:

- v2.7 / dataVersion 3
- v2.6 / dataVersion 2
- older legacy state through migration logic

The user should still export a JSON backup before switching downloaded HTML versions because `file://` browser storage behavior may vary by browser/device.

---

# 12. Automated/static QA

Passed:

- JavaScript syntax check
- 2,866 cards retained
- 2,866 unique card IDs
- no duplicate static HTML IDs
- Sentence tab retained
- Word detail/library retained
- TTS/example structures retained
- old fixed `todayTargets()` scheduler removed
- old visible `dailyNew` and `dailyReview` controls removed
- new Smart Session functions present
- separate Today/Custom session slots present

---

# 13. Runtime logic QA

Passed test scenarios:

- brand-new learner receives New introduction
- Seen card receives meaning test rather than intro
- Due card is selected for review
- stale Today retry invalidates after a later attempt in another mode
- Today and Custom active sessions can coexist
- old fixed Today session queue retires during v2 migration
- old card progress remains intact
- old Custom active session is retained
- old `dailyNew=30` maps to `newPace=high`
- stale Custom intro resolves using latest shared progress
- finite Custom session still starts with intro for a genuinely New card
- Custom intro → meaning → secondary flow still works
- finite Custom target still completes

---

# 14. Runtime Smart Session distribution checks

These are not fixed quotas. They are sanity checks showing the selector adapts.

Representative 10-interaction simulations:

## brand new
approximately:
- 5 New intros
- 4–5 first/secondary learning tests
- occasional retry

## 100 Due, no Seen
approximately:
- 7–8 Due
- about 1 New
- about 1 New-learning test
- occasional retry

## 20 Due + 100 Seen
approximately:
- 4–5 Due
- 3–4 Seen/learning tests
- very little New
- retries as needed

## 10 Weak + 50 Due
approximately:
- about 5–6 Weak
- about 2–3 Due
- retries as needed
- almost no New during a short 10-interaction run

Representative 100-interaction brand-new run:

- about 30 New introductions
- about 55 first/secondary learning tests
- about 14 retries

With extremely large backlog such as:

- Weak 20
- Due 300
- Seen 50

the normal pace can temporarily introduce **zero New words in the first 100 interactions**.

That is accepted behavior in v2.7: the app prioritizes clearing learning debt before adding more unfinished material.

---

# 15. Known limitations intentionally left for later

v2.7 does NOT yet solve:

- multiple simultaneous Weak skills
- global `longReviewPassed` Mastered weakness
- Weak / Hard / ★ semantic separation
- reverse-question synonym ambiguity
- second-generation distractor quality
- accent-insensitive library search
- in-app `검토 필요` content flag
- transactional answer undo

These remain future work and should not be confused with v2.7 regressions.

---

# 16. GUI QA limitation

Full browser/Android GUI automation was not available in the development environment.

Therefore final real-device checks are still needed for:

- Today button/start/resume feel
- Custom resume card
- Settings `새 단어 속도`
- long Smart study flow
- Android back/reload behavior
- TTS during Smart Session
- direct-open storage migration on the user’s actual browser

The logic and static structure passed automated tests, but real Android use is still the final UX validation.
