# TEF Vocab Loop

개인용 TEF Canada 프랑스어 어휘 학습 앱입니다.

현재 기준 버전: **v2.9.2**

앱은 single-file HTML 구조이며 `index.html` 하나만으로 실행됩니다.

## 현재 규모

- 2,866 French vocabulary cards
- A1-A2 + B1-B2
- Korean / English study language
- shared progress across both language modes
- Smart Session
- Custom Study
- Meaning / Listening / Reverse / Spelling
- Word library
- Sentence library
- TTS + slow TTS
- streak calendar
- backup / restore
- v2.9 example-quality project: 612 examples rewritten so far

## GitHub Pages

이 repository를 GitHub에 올린 뒤:

`Settings → Pages → Deploy from a branch → main / root`

를 선택하면 `index.html`을 앱으로 사용할 수 있습니다.

중요: 로컬 `file://` HTML과 GitHub Pages 주소는 저장 공간(origin)이 다릅니다.  
기존 학습 기록이 있다면 **기존 앱에서 JSON backup을 먼저 export**하고, GitHub Pages 버전에서 필요하면 import하세요.

## 다음 세션에서 이어서 작업할 때

가장 먼저 읽을 파일:

**`NEW_SESSION_START_HERE.md`**

그 안에 현재 구조, 사용자의 제품 결정, 최근 변경, 예문 개선 현황, 다음 작업 순서가 정리되어 있습니다.

## Repository 구조

- `index.html` — GitHub Pages용 최신 앱
- `TEF_Vocab_Loop_v2_9_2.html` — 최신 버전 스냅샷
- `NEW_SESSION_START_HERE.md` — 새 ChatGPT 세션용 핵심 handoff
- `NEXT_SESSION.md` — 다음 작업 빠른 요약
- `docs/` — 설계/히스토리/QA/예문 변경 기록
- `archive/` — 이전 앱 버전

## 현재 진행 중인 작업

예문 품질 개선.

단순히 타깃 단어를 넣은 문장이 아니라:

- 실제 쓰임
- collocation
- 전치사 구조
- 함께 쓰는 동사
- TEF에서 재활용 가능한 표현

을 보여주는 문장을 목표로 합니다.

이미 좋은 예문은 유지하고, 저가치 문장만 단계적으로 교체합니다.
