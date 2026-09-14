# TEF Vocab Loop v2.7.2 — QA Report

Numeric-display patch verification:

- PASS — JavaScript syntax
- PASS — 2,866 cards retained
- PASS — unique card IDs
- PASS — all 58 true number cards mapped
- PASS — non-number category neighbors excluded
- PASS — number distractors restricted to number cards
- PASS — quiz answer comparison uses display layer
- PASS — original source Korean is untouched
- PASS — 20 is shown for vingt
- PASS — 1,000 is shown for mille
- PASS — 100,000,000,000 is shown for cent milliards

Design confirmation:
- The embedded PDF-derived `ko` meanings are unchanged.
- Only the UI presentation layer changes true number-value cards to Arabic numerals.
- `le moment`, `le nombre`, `le numéro`, and `le chiffre` remain ordinary Korean-meaning vocabulary.
- Meaning/listening choices, reverse prompts, spelling prompts, intro, feedback, and library display use the numeral layer.

Examples:
- `un` → `1`
- `vingt et un` → `21`
- `quatre-vingts` → `80`
- `cent un` → `101`
- `mille` → `1,000`
- `million` → `1,000,000`
