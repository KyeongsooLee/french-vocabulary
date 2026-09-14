# TEF Vocab Loop v2.9.0 — Example Quality Upgrade QA

## Scope
- Examples changed: **286**
- This is the first large content-quality pass, focused on the most obviously low-value A1-A2 templates.
- French example, Korean translation, and English translation were updated together.
- Vocabulary meanings and source card identity were not changed.

## Automated checks
- PASS — JavaScript syntax
- PASS — 2,866 cards retained
- PASS — 2,866 English records retained
- PASS — stable card IDs
- PASS — vocabulary/source fields unchanged
- PASS — replacement count >= 280
- PASS — all changed cards have Korean translation
- PASS — all changed cards have English translation
- PASS — low-value J'aime templates removed
- PASS — C'est template count materially reduced

## Repetitive-template reduction
- `J'aime ...` examples: 192 → 3
- `C'est ...` examples: 209 → 116
- exact duplicate French example strings remaining in full corpus: 14

## Content focus in this pass
- 100 food/meal cards: practical cooking, ordering, storage, serving, and preparation contexts.
- 31 accessories cards: wearing, carrying, storing, and real-use contexts.
- 19 sports cards: natural `jouer à / faire de` usage and frequency/context.
- 19 clothing cards: dressing for weather/work/activities.
- 19 drinks cards: ordering, serving, and consumption contexts.
- 1 instrument card.
- 57 A1-A2 adjective cards: replaced one-line `C'est ...` examples with contextual structures.
- 23 A1-A2 ADVERBES entries with weak/malformed `C'est ...` examples.
- 17 A1-A2 OBJETS entries with weak `C'est ...` examples.

## Known next content work
- B1-B2 `L’adjectif` still contains many `C'est ...` placeholders and needs a careful second pass.
- Repetitive templates such as `On parle souvent de ... dans les médias`, `Nous passons près de ...`, and generic medical/location templates remain candidates.
- English translations in the rest of the corpus remain bulk-generated; this pass manually aligns the changed examples only.

## QA caveat
- Automated checks can catch structure, missing translations, duplicates, and some target-word omissions.
- Naturalness and pedagogical value are ultimately semantic; real-study feedback remains important.

## Target-presence heuristic warnings
- 2 changed examples did not contain an obvious lexical token from the display form. Some are legitimate inflection/phrase cases and should be manually reviewed.
  - v1104 `le déjeuner` → `Je déjeune vers midi quand je travaille.`
  - v1105 `le dîner` → `Nous dînons ensemble après le travail.`
