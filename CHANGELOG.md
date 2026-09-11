# TEF Vocab Loop — CHANGELOG

This changelog is intentionally concise. See `TEF_Vocab_Project_History.md` for reasoning and detailed feedback.

## Pre-v2
- Built earlier flashcard/PWA-style vocabulary app.
- Confirmed direct-open HTML works on Android.
- Shifted toward a 말해보카-inspired micro-quiz learning loop.

## v1 / first Loop build
- French→Korean multiple choice.
- Korean→French multiple choice.
- audio→meaning.
- spelling input.
- delayed error reappearance.
- TTS.
- hard/starred words.
- IndexedDB.
- JSON backup.

## v2.0
- Rebuilt learning engine around separate skills:
  - meaning
  - listening
  - reverse
  - spelling
- Added state-based learning model.
- Added delayed retries and dynamic task selection.
- Added time-separated review stages.
- Spelling made non-mandatory for mastery.
- Added confusable distractor data.
- Added richer dashboard/status data.

## v2.1
- Added example sentence + Korean translation for all 2,866 cards.
- Added normal/slow sentence TTS.
- Added per-card TTS text separate from display text.
- Removed spoken grammar metadata such as `(f.)` / `(m.)`.
- Improved example quality and removed previous safe-placeholder examples.
- Refined answer-feedback UI.
- Removed redundant system-like feedback text.
- Iterated on word-audio placement:
  - first moved it into the example block,
  - then separated it back out visually after user feedback.
- Final layout: word audio in feedback area; example alone in white card.

## v2.2
- Fixed whole-topic custom study source-order bias.
- Added category-balanced selection while preserving priority.
- Added safe previous-new-word replay/back control.
- Replay does not roll back learning state.

## v2.3
- Added `Seen` state.
- `확인했어요` alone no longer counts as Learning.
- First actual quiz response begins Learning.
- Previously Seen unfinished words resume from a meaning test.
- Added Seen to dashboard.
- Mobile status grid adjusted for 6 states.

## v2.4
- Turned Word tab into a vocabulary library.
- Added mobile bottom-sheet word detail.
- Added word/example TTS inside detail.
- Added all four skill bars.
- Added last/next review info and stats.
- Added star toggle in detail.
- Removed PDF page/source from visible Word-tab UI.

## v2.5
- Added resumable active study sessions.
- Home shows in-progress study with continue/start-over.
- Manual exit persists session.
- Completion clears active session.
- Original focus mode survives resume.
- Active study time avoids counting long app-closed periods.
- Performed targeted spelling QA.
- Confirmed 2,166 / 2,866 cards currently spelling-eligible.

## v2.6
- Added Sentence tab.
- Bottom navigation is now:
  - 오늘
  - 학습
  - 단어
  - 문장
  - 설정
- Added Recommended / All sentence views.
- Added sentence search, level/category filters.
- Added normal/slow sentence TTS.
- Added link to associated word detail.
- Added heuristic recommendation scoring for reusable learning value.
- v2.6 recommended set:
  - 353 total
  - 78 A1-A2
  - 275 B1-B2
  - 269 curated
  - 84 reviewed
