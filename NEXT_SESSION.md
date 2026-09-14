# NEXT SESSION

Current baseline: **TEF Vocab Loop v2.9.2**

Read first:
1. `NEW_SESSION_START_HERE.md`
2. `docs/PROJECT_HANDOFF.md`
3. `docs/TEF_Vocab_Project_History.md`
4. `docs/CHANGELOG.md`
5. `docs/QA_NOTES.md`
6. `docs/reports/V2_9_2_EXAMPLE_QA_REPORT.md`
7. `index.html`

## Immediate active project

Continue the **Example Quality Upgrade**.

Cumulative rewrites: **612 / 2,866**.

Next high-confidence repetitive families in v2.9.2:

- `Ce cours porte sur ...` — 34
- `Ce documentaire parle de/du ...` — 28
- `Le journal parle de/du ...` — 24
- `Ce livre parle de/du ...` — 26
- `Le médecin parle de/du ...` — 32

Total: **144**.

For every changed example, update French + Korean + English together.

Do not mass-regenerate all 2,866 examples. Preserve already-good examples.

After obvious template families are cleaned, switch to semantic review for individually low-value examples.

## Critical invariants

- 2,866 stable cards.
- One shared learner progress for Korean/English modes.
- Seen ≠ Learning.
- Spelling is reinforcement, not mandatory for mastery.
- Today = continuous Smart Session, no fixed daily quota.
- Custom Study = finite, but shares global progress.
- Calendar current-day class is `isToday`, not generic `today`.
- Keep word-audio controls outside the white example card.
- Keep original source meanings unless the user explicitly asks to revise them.
