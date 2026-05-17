# 노시언(Nosion) 위젯 디자인 시스템

> 노션(Notion) 임베드용 미니멀 위젯 시리즈의 공통 디자인 가이드.
> **새 위젯을 만들거나 외부에서 가져온 위젯을 통합할 때 이 문서를 따라주세요.**

---

## 0. 핵심 원칙 (TL;DR)

1. **미니멀 모노톤** — 회색 스펙트럼만 사용, 채도/액센트 색상 지양
2. **단일 HTML 파일** — vanilla HTML/CSS/JS, 외부 의존성은 Pretendard 폰트 하나만
3. **노션 임베드 친화** — iframe 안에서 단독 위젯으로 동작, 외부 통신 없음
4. **카드 1개 = 위젯 1개** — 모든 위젯은 동일한 카드 구조
5. **드래그/리사이즈 가능** — 위치·크기는 사용자가 조절, localStorage에 영구 저장
6. **호버 인터랙션** — 컨트롤·핸들은 호버 시에만 보여 깔끔함 유지

---

## 1. CSS 디자인 토큰

모든 위젯의 `:root`에 반드시 포함:

```css
:root {
  --background: hsl(0 0% 100%);          /* 페이지 배경 */
  --foreground: hsl(220 10% 55%);        /* 메인 텍스트 — 회색 */
  --card: hsl(0 0% 100%);                /* 카드 배경 */
  --border: hsl(220 9% 84%);             /* 보더 */
  --muted: hsl(210 20% 96%);             /* 호버 배경, 미묘한 영역 */
  --muted-foreground: hsl(215 14% 50%);  /* 보조 텍스트, 라벨 */
  --radius: 0.75rem;                     /* 카드 둥근 모서리 */
  --transition: 0.2s ease;               /* 기본 트랜지션 */
}
```

**중요**: `--foreground`는 `hsl(220 10% 55%)` (미디엄 그레이)입니다. 검정에 가까운 색은 일관성을 깨므로 피하세요.

---

## 2. 폰트 & 타이포그래피

### 폰트 로드 (head)

```html
<link rel="stylesheet" as="style" crossorigin
  href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css" />
```

### body 설정

```css
html, body {
  font-family: 'Pretendard Variable', 'Pretendard', -apple-system,
               BlinkMacSystemFont, system-ui, 'Segoe UI', sans-serif;
  background: var(--background);
  color: var(--foreground);
  margin: 0;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  word-break: keep-all;     /* 한글 줄바꿈 자연스럽게 */
}
```

### 스타일 규칙
- **숫자**: `font-variant-numeric: tabular-nums` 적용 (자릿수 정렬)
- **letter-spacing**: 큰 글씨엔 -0.02em ~ -0.04em (시각 보정), 작은 라벨엔 0.02em ~ 0.1em (가독성)
- **font-weight**: 본문 400~500, 강조 500, 굵게는 600+ 지양 (모노톤 톤 유지)

### 텍스트 계층 예시

| 용도 | 크기 | 색상 |
|------|------|------|
| 큰 숫자/시간 | 2.5rem ~ 4.5rem | `var(--foreground)` |
| 위젯 타이틀 | 1.125rem ~ 1.25rem | `var(--foreground)` |
| 본문 | 0.8125rem ~ 0.875rem | `var(--foreground)` |
| 라벨 (UPPERCASE) | 0.6875rem | `var(--muted-foreground)` |
| 메타/푸터 | 0.6875rem | `var(--muted-foreground)` + opacity 0.5~0.7 |

---

## 3. 카드 구조 (필수 마크업)

```html
<body>
  <div class="widget-wrapper" id="widget">
    <div class="card" id="card">

      <!-- 위젯 콘텐츠 -->

      <!-- 4방향 리사이즈 핸들 (필수) -->
      <div class="resize-handle handle-top" data-direction="top"><div class="bar"></div></div>
      <div class="resize-handle handle-bottom" data-direction="bottom"><div class="bar"></div></div>
      <div class="resize-handle handle-left" data-direction="left"><div class="bar"></div></div>
      <div class="resize-handle handle-right" data-direction="right"><div class="bar"></div></div>
    </div>

    <!-- 우측 컨트롤 박스 (필수) -->
    <div class="controls">
      <button class="control-btn" id="resizeResetBtn"
              data-tooltip="크기 초기화" title="크기 초기화"
              type="button" aria-label="카드 크기 초기화">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <polyline points="15 3 21 3 21 9"/>
          <polyline points="9 21 3 21 3 15"/>
          <line x1="21" y1="3" x2="14" y2="10"/>
          <line x1="3" y1="21" x2="10" y2="14"/>
        </svg>
      </button>
      <!-- 추가 컨트롤 버튼이 있다면 여기에 -->
    </div>
  </div>
</body>
```

### 카드 기본 스타일

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  padding: 1rem;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.widget-wrapper {
  display: flex;
  align-items: flex-start;
  gap: 0.5rem;
  position: relative;
  width: fit-content;
}

.card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 1.75rem 2rem 1.5rem;   /* 위 1.75 좌우 2 아래 1.5 */
  width: 360px;                    /* 기본 너비 */
  position: relative;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: none;                /* 그림자 없음 */
}
```

**카드 너비**: 기본 360px. 위젯 성격에 따라 320~400px 범위에서 조정.

---

## 4. 인터랙션 패턴

### 4.1 드래그로 위치 이동

- **드래그 영역**: 카드 빈 영역 전체 (인터랙티브 요소만 제외)
- **블록 셀렉터**: `.control-btn`, `.resize-handle`, `button`, `[contenteditable]`, `a`, `input`
- 헤더가 있는 위젯이면 헤더에만 한정해도 OK (라벨 클래스: `.header`)

### 4.2 4방향 리사이즈

- 4개 핸들: 위/아래/좌/우
- **호버 시에만** `.bar`가 보임 (opacity 0 → 1)
- 호버한 핸들은 더 두꺼워짐 (1px → 3px)
- 최소/최대 크기 제한 필수

### 4.3 우측 컨트롤 박스

- 평소 opacity 0.25 (살짝 보임)
- `.widget-wrapper:hover .controls` 호버 시 opacity 1
- **크기 초기화 버튼은 항상 첫 번째**
- 추가 액션 버튼 (예: 리셋, 설정)은 그 아래

### 4.4 커스텀 툴팁

`data-tooltip` 속성으로 정의. CSS `::before/::after` 의사요소로 렌더링 (외부 라이브러리 X).

---

## 5. localStorage 키 규칙

**키 형식**: `nosion-{widget-name}-{purpose}-v1`

| 용도 | 키 예시 |
|------|--------|
| 위젯 데이터 | `nosion-moon-phase-v1` |
| 카드 크기 | `nosion-moon-phase-size-v1` |
| 카드 위치 | `nosion-moon-phase-pos-v1` |
| 위젯 설정 | `nosion-moon-phase-city-v1` |

**필수**: 모든 위젯은 크기·위치를 `size`/`pos` 키로 저장하고 복원해야 함.

---

## 6. 색상 사용 가이드

| 용도 | 권장 색상 |
|------|---------|
| 메인 텍스트 | `var(--foreground)` |
| 보조 텍스트, 라벨 | `var(--muted-foreground)` |
| 매우 옅은 텍스트 | `hsl(220 10% 65%)` |
| 약간 더 진한 강조 | `hsl(220 10% 42%)` ~ `hsl(220 13% 35%)` |
| 보더 | `var(--border)` |
| 호버 배경 | `var(--muted)` |

**금지**: 채도 있는 색상 (빨강·파랑·녹색 등), 검정에 가까운 색 (`hsl ... < 30%`), 그라데이션 (단, 입체감을 위한 미묘한 그레이 라디얼 그래디언트는 OK)

---

## 7. 참고 위젯 (Reference Widgets)

코드 패턴과 톤을 확인할 때 다음 파일들을 참고하세요:

| 위젯 | 특징 |
|------|------|
| `moon-phase.html` | 가장 정제된 예시. SVG 시각화 + 드롭다운(도시 선택) + 데모 모드 |
| `lunch-picker.html` | 애니메이션 (3D 원통 회전) + 동적 콘텐츠 (메뉴 추가/삭제) + confetti 효과 |
| `routine-tracker.html` | 그리드 레이아웃 + 그리드 셀 인터랙션 + 트렌드 차트 |

새 위젯을 만들 때 가장 비슷한 패턴의 위젯 코드를 베이스로 시작하는 게 가장 빠릅니다.

---

## 8. 체크리스트 (위젯 완성도 점검)

새 위젯을 만들거나 외부에서 가져왔을 때 다음을 모두 통과해야 합니다:

- [ ] `:root`에 8개 CSS 토큰 정의됨 (위 1번 섹션)
- [ ] Pretendard Variable 폰트 로드, `word-break: keep-all`
- [ ] `.widget-wrapper > .card + .controls` 구조
- [ ] 4방향 `.resize-handle` (호버 시만 보임)
- [ ] 우측 `.controls`에 크기 초기화 버튼
- [ ] 카드 드래그로 위치 이동 가능 (인터랙티브 요소 제외)
- [ ] localStorage에 크기·위치 저장 (`nosion-{name}-{size|pos}-v1`)
- [ ] 모든 `<button>`에 `type="button"` 지정
- [ ] 검정 텍스트 없음 (모두 `var(--foreground)` = hsl 220 10% 55% 또는 더 옅은 톤)
- [ ] 채도 있는 색상 없음 (회색 스펙트럼만)
- [ ] 그림자 없음 (`box-shadow: none`)
- [ ] 외부 라이브러리 없음 (Pretendard CDN 폰트만 허용)

---

## 9. Lovable / 외부 AI 도구에 줄 수 있는 컨텍스트

이 문서를 통째로 시스템 프롬프트나 프로젝트 컨텍스트로 넣으세요.
추가로 다음 한 줄을 강조하면 일관성이 높아집니다:

> "Reference one of these widgets in the same repo for code patterns and tone:
> `moon-phase.html`, `lunch-picker.html`, `routine-tracker.html`. Match their
> CSS token usage, card structure, resize/drag implementation, and Korean
> typography conventions exactly."

---

_Last updated: 2026-05_
