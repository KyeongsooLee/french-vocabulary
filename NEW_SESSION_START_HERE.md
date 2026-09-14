# NEW SESSION — START HERE

## 현재 기준 버전

**TEF Vocab Loop v2.9.2**

새 세션에서는 `index.html` 또는 `TEF_Vocab_Loop_v2_9_2.html`을 최신 기준으로 사용한다.

현재 핵심 데이터:

- 프랑스어 학습 카드: **2,866개**
- 원본 A1-A2 / B1-B2 어휘 기반
- 한국어 + 영어 이중 언어 UI/뜻/예문
- 한국어/영어 모드 사이에 **하나의 공용 학습 진행도**
- v2.9 예문 품질 개선 누계: **612개**
  - v2.9.0: 286개
  - v2.9.1: 167개
  - v2.9.2: 159개
- 카드 ID, 원본 단어 뜻, 학습 진행 데이터 구조는 예문 개선 과정에서 유지됨

---

## 다음 세션에서 가장 먼저 할 일

현재 사용자가 진행 중인 작업은 **예문 품질 개선**이다.

다음 순서는 우선 아래의 명백한 반복 템플릿을 실제 학습 가치가 있는 문장으로 교체하는 것이다.

현재 v2.9.2 기준 후보 수:

- `Ce cours porte sur ...` — **34개** (`L’histoire`)
- `Ce documentaire parle de/du ...` — **28개** (`L'environnement`)
- `Le journal parle de/du ...` — **24개** (`Les actualités`)
- `Ce livre parle de/du ...` — **26개** (`La religion`)
- `Le médecin parle de/du ...` — **32개** (`La santé`)

이 다섯 계열만 합쳐도 **144개**다.

그 다음 후보:

- `On parle souvent du ...` — **23개**
- `On parle de ...` — **93개**

단, 후자의 `On parle de ...` 계열은 모두 나쁜 문장이라고 가정하면 안 된다.  
이제부터는 **패턴 자체보다 문장의 학습 가치**를 보고 개별 판단해야 한다.

### 예문 품질 기준

좋은 예문은 단순히 타깃 단어를 포함하는 문장이 아니다.

가능하면 다음 중 하나 이상을 가르쳐야 한다.

- 실제 생활에서 자주 쓰는 상황
- 동사 + 전치사 구조
- 자연스러운 collocation
- 해당 명사와 자주 같이 쓰는 동사
- 재사용 가능한 B1-B2 표현
- TEF 말하기/쓰기에서 활용할 수 있는 문장 구조
- 단어의 의미 차이를 드러내는 문맥

피해야 할 것:

- `J'aime X.`
- `C'est X.`
- `On parle de X.`
- `Ce livre parle de X.`
- 단어를 문장에 억지로 넣기만 한 예시
- 사전 정의를 문장처럼 바꾼 것
- 지나치게 희귀하거나 과하게 어려운 문장
- 프랑스어는 자연스럽지만 실제 학습에는 남는 것이 없는 문장

**기존 예문이 이미 좋다면 건드리지 않는다.**

프랑스어 예문을 바꿀 때는 항상 세 항목을 같이 맞춘다.

1. `exFr`
2. `exKo`
3. English sidecar example `e`

원본 단어 뜻 `ko`, 영어 단어 뜻, 카드 ID는 예문 작업에서 함부로 수정하지 않는다.

---

## 지금까지 제거한 저가치 예문 계열

### v2.9.0 — 286개

주요 작업:

- `J'aime ...` 대량 템플릿 제거  
  192개 → 3개
- 음식 / 스포츠 / 옷 / 액세서리 / 음료 예문 실용화
- A1-A2 형용사의 단순 `C'est ...` 예문 대량 개선
- 음식은 가능하면 실제 조리 동사/콜로케이션을 사용

예:

- `les lunettes`  
  → `Je porte des lunettes pour lire les petits caractères.`
- `l'épinard`  
  → `J'ajoute des épinards à la poêle à la fin de la cuisson.`
- `difficile`  
  → `Il est difficile de trouver un logement abordable dans cette ville.`

### v2.9.1 — 167개

완전히 제거:

- B1-B2 `L’adjectif` 단순 `C'est ...` 80개
- `On parle souvent de X dans les médias.` 34개
- `Nous passons près de X.` 53개

예:

- `fier`  
  → `Je suis fier d'avoir terminé ce projet à temps.`
- `impropre`  
  → `Cette eau est impropre à la consommation.`
- `l'asile`  
  → `Il a demandé l'asile après avoir fui son pays.`

### v2.9.2 — 159개

완전히 제거:

- `Nous passons près du ...` 45개
- `Je parle souvent avec ...` 30개
- `Le médecin examine ...` 45개
- `On voit souvent ... dans la nature.` 39개

학습 방향:

- 장소 → 길찾기 / 용무 / 영업시간 / 이동
- 가족·사람 → 실제 관계와 생활 상황
- 신체 → 통증 / 부상 / 위생 / 일상 표현
- 자연 → 특징적인 동사와 자연스러운 collocation

예:

- `le genou`  
  → `Mon genou me fait mal quand je monte les escaliers.`
- `la racine`  
  → `Les racines absorbent l'eau dans le sol.`

---

## 앱의 핵심 학습 구조

### 상태

- New
- Seen
- Learning
- Familiar
- Mastered
- Weak

중요:

> **Seen ≠ Learning**

새 단어 소개만 보고 나간 카드는 Seen이다.  
실제 핵심 문제를 풀어야 Learning으로 넘어간다.

### 스킬

카드마다 별도 스킬:

- `meaning`: French → meaning
- `listening`: audio → meaning
- `reverse`: Korean/English → French
- `spelling`: typed French

`spelling`은 일반 자동 학습에 등장할 수 있지만 **Mastered 필수 조건은 아니다.**

### 복습 철학

대략:

same session → 1d → 3d → 7d → 14d → 30d → 60d

같은 세션에서 여러 번 맞힌 것보다 **시간 간격을 둔 성공**이 더 중요하다.

---

## Today Study vs Custom Study

매우 중요한 불변 규칙:

> Today Study와 Custom Study는 서로 다른 세션이지만, 같은 하나의 global learner record를 사용한다.

### Today Study

- Continuous Smart Session
- 하루 목표 개수 없음
- 10개만 하고 끝내도 되고 100개 해도 됨
- Due / Seen / Weak / Retry / New를 현재 상태에 맞게 동적으로 선택

### Custom Study

- 사용자가 level / category / size / focus를 지정
- finite session

Custom에서 단어 상태가 바뀌면 Today도 그 최신 global state를 사용해야 한다.

---

## Smart Session 관련 중요한 결정

- 실패한 카드 재시험은 약 3–6문제 뒤
- 한 카드/스킬당 pending retry는 하나
- New를 영원히 막으면 안 됨
- backlog가 많으면 새 단어 유입 속도만 줄임
- whole-topic custom study는 source order가 아니라 **category-balanced**
- accidental intro skip용 뒤로가기 버튼은 **이전 소개 재표시**일 뿐 state rollback이 아님

---

## 철자 문제

현재 철자 문제:

- 일반 텍스트 입력
- `_ _ _` 형태 힌트 없음
- 기본 상태에서는 힌트 없음
- `힌트 보기`를 누르면 시작 부분만 표시
- 힌트를 사용해 맞힌 경우 정답 처리는 되지만 spelling step은 올리지 않음
- 복잡한 프랑스어 표현은 spelling 자동 출제에서 제외
- spelling은 mastery 필수가 아님

---

## 숫자 카드 표시

현재 규칙:

- 1–99 → 숫자 (`1`, `21`, `80`)
- 100+:
  - 한국어 모드 → `백`, `천`, `백만`, `억` 등
  - 영어 모드 → `one hundred`, `one thousand`, `one million` 등

숫자 카드 내부 정체성 자체는 전체 58개 숫자 카드로 유지되어야 한다.

---

## 한국어 / 영어 이중언어 구조

하나의 프랑스어 corpus를 공유한다.

언어 전환 시 바뀌는 것:

- UI
- 단어 뜻
- 예문 번역
- 문제 prompt / choices
- category 표시
- Word library
- Sentence library
- search

진행도는 절대 언어별로 나누지 않는다.

English sidecar:

- 2,866 meanings
- 2,866 example translations
- stable card ID 기준
- 구조적으로 완전함
- 그러나 전체가 한 줄씩 사람에게 검수된 것은 아님
- v2.9에서 바뀐 예문들의 영어 번역은 해당 변경과 함께 맞춰짐

---

## Word / Sentence 탭

하단 탭:

**오늘 · 학습 · 단어 · 문장 · 설정**

### Word

단어를 누르면:

- level / category / status
- French + meaning
- word TTS
- example + translation
- example TTS
- 4 skill bars
- current stage
- next review
- last study
- correct/wrong
- ★ / hard

PDF 원본 페이지 표시는 의도적으로 UI에서 제거됨.

### Sentence

- Recommended / All
- search
- level/category filter
- normal/slow TTS
- linked word detail

추천 문장은 “좋은 예문 전체”가 아니라 **문장 자체를 따로 외울 가치가 높은 것**을 의미한다.

현재 추천 heuristic은 아직 완벽하지 않다.

---

## TTS 불변 규칙

화면 표시 문자열과 읽는 문자열은 분리한다.

예:

- display: `l'expression (f.)`
- speech: `l'expression`

문법 메타데이터:

- `+ inf`
- `+ sub`
- `+ cond`
- `qn`
- `qc`

같은 것은 TTS가 읽으면 안 된다.

하지만 `(bel)` 같은 실제 lexical variant까지 무작정 삭제하면 안 된다.

---

## 캘린더 관련 최근 수정

### v2.8.3
상단 streak pill을 누르면 월간 학습 캘린더 표시.

### v2.8.4
모바일에서 달력 원이 지나치게 커지고 overflow되는 문제 해결.

핵심:

- grid cell과 보이는 date circle 분리
- `.calendarDay` = 배치
- `.calendarDate` = 작은 원
- `aspect-ratio:1/1`을 전체 grid cell에 다시 적용하지 말 것
- `repeat(7,minmax(0,1fr))`
- 좁은/낮은 화면 media rule
- modal viewport max-height + internal scroll

### v2.8.5
오늘 날짜 표시:

- 오늘 + 아직 학습 안 함 → 속이 빈 중립색 원 + 오늘 숫자 accent
- 오늘 + 학습함 → 보라색 채운 원 + 오늘 숫자 accent 유지

### v2.8.6
중요한 버그 수정:

달력 오늘 날짜에 `today` 클래스를 사용했더니 홈 화면 `.today` CSS와 충돌해서 오늘 원만 아래로 내려갔다.

현재 달력은 반드시:

`calendarDay isToday`

를 사용한다.

**달력에 generic `today` 클래스를 다시 쓰지 않는다.**

---

## 아직 남은 학습 엔진 구조적 과제

예문 프로젝트가 끝난 뒤 우선순위가 높다.

1. **Multi-skill Weak**
   - 현재 `weakSkill` 하나뿐이라 listening + reverse가 동시에 약한 상태를 제대로 표현하지 못함.

2. **Mastered evidence**
   - `longReviewPassed`가 전역이라 스킬별 증거가 충분하지 않을 수 있음.

3. **Weak / Hard / ★ 의미 분리**
   - 현재 hard 판정에는 현재 Weak, star, 역사적 오류율이 섞여 있음.

4. **Reverse ambiguity**
   - 한국어/영어 뜻 하나에 여러 French 답이 가능한 경우 공정하지 않은 문제가 생길 수 있음.

5. **Distractor 개선**

이 작업은 예문 변경과 한 번에 같이 하지 않는 것을 권장한다.

---

## 앞으로 좋은 제품 개선 후보

예문 품질 개선 후:

- Word library 상태 필터
  - New / Seen / Learning / Familiar / Mastered / Weak / ★ / 검토 필요
- sorting
- accent-insensitive search
- 카드별 `⚑ 검토 필요`
  - 뜻 / 예문 / TTS / 보기 / 기타
- Sentence 추천 태그/curation 개선
- backup/import robustness
- explicit schema migration
- backup reminder
- 이후에야 로그인/랭킹/클라우드 등을 고려

---

## UI/제품 취향 — 반드시 유지

- 현대적이고 compact한 모바일 UI
- 쓸데없는 grammar/game/conversation 기능 붙이지 않음
- pronunciation/TTS 매우 중요
- word audio와 sentence audio 모두 필요
- example은 정답을 맞힌 **후에** 표시
- answer feedback의 word audio를 example white card 안으로 합치지 않음
- bottom nav text는 작고 compact하게
- Today에 고정 일일 quota를 다시 넣지 않음
- 실제 폰 사용에서 발견되는 미세한 레이아웃 문제도 중요하게 취급

---

## 파일 구조

GitHub 저장소에서는:

- `index.html` — GitHub Pages용 최신 앱
- `TEF_Vocab_Loop_v2_9_2.html` — 최신 버전 스냅샷
- `docs/PROJECT_HANDOFF.md`
- `docs/TEF_Vocab_Project_History.md`
- `docs/CHANGELOG.md`
- `docs/QA_NOTES.md`
- `docs/ROADMAP_NEXT.md`
- `docs/reports/` — QA 기록
- `docs/example-audits/` — v2.9 예문 변경 전/후 TSV
- `archive/` — 이전 HTML 버전

---

## GitHub Pages로 옮길 때 매우 중요

현재 사용자는 Android에서 다운로드한 HTML을 직접 여는 방식도 사용했다.

`file://`로 열던 앱과 `https://...github.io/...`로 여는 앱은 브라우저 입장에서 **다른 origin**이다.

따라서 진행도가 자동으로 따라간다고 가정하면 안 된다.

안전한 순서:

1. 기존 앱에서 JSON backup export
2. 기존 HTML 파일 보관
3. GitHub Pages의 `index.html` 열기
4. 진행도가 없다면 JSON backup import
5. 카드 수 / streak / 최근 학습 상태 확인

GitHub에 올렸다고 기존 로컬 HTML을 바로 삭제하지 않는다.

---

## 새 세션에서 읽을 순서

1. `NEW_SESSION_START_HERE.md`
2. `docs/PROJECT_HANDOFF.md`
3. `docs/TEF_Vocab_Project_History.md`
4. `docs/CHANGELOG.md`
5. `docs/QA_NOTES.md`
6. 최신 관련 QA report
7. `index.html`

새 세션에서 사용자가 **“예문 개선 계속하자”**라고 하면,
우선 v2.9.2에서 위에 정리한 144개 고확신 템플릿 후보부터 검토한다.

그 다음에는 단순 prefix 탐색을 넘어,
2,866개 전체를 의미적으로 평가해서 **겉보기에는 정상인데 학습가치가 낮은 예문**을 찾는 단계로 넘어간다.
