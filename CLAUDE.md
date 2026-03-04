# Bible Random — 프로젝트 명세

## 개요

개역한글 성경에서 주제별 말씀을 랜덤으로 보여주는 **단일 HTML 파일** 웹앱.
GitHub Pages로 배포되며 별도 빌드/서버 없이 동작한다.

- **GitHub**: https://github.com/greatacme/bible_random
- **Live**: https://greatacme.github.io/bible_random/
- **작업 디렉터리**: `C:\Users\Public\dev\bible-random` (WSL: `/mnt/c/Users/Public/dev/bible-random`)

## 파일 구조

```
bible-random/
├── index.html    # 앱 전체 (HTML + CSS + JS 단일 파일)
├── README.md     # GitHub 프로젝트 설명
└── CLAUDE.md     # 이 파일 (Claude Code 컨텍스트)
```

## 기술 스택

- **프레임워크**: 없음. 순수 HTML/CSS/JS
- **외부 의존성**: Google Fonts (Noto Serif KR) — CDN 로드
- **API**: bible.helloao.org (무료, 키 불필요, CORS 허용)
- **호스팅**: GitHub Pages (main 브랜치 루트)

## API 상세

- **Base URL**: `https://bible.helloao.org/api/kor_old`
- **번역본**: `kor_old` — 개역한글 (Korean Revised Version, 1961), 퍼블릭 도메인
- **엔드포인트**: `GET /api/kor_old/{BOOK_ID}/{CHAPTER}.json`
  - 예: `/api/kor_old/PSA/23.json` → 시편 23장
- **응답 구조**:
  ```json
  {
    "chapter": {
      "number": 23,
      "content": [
        { "type": "verse", "number": 1, "content": ["여호와는 나의 목자시니..."] },
        ...
      ]
    }
  }
  ```
- 절(verse)의 `content`는 문자열 배열 — `typeof c === 'string'`으로 필터링하여 결합

### 한국어 성경 저작권 참고

현대 번역본(개역개정, 새번역/RNKSV, 공동번역)은 대한성서공회 저작권으로 무료 API 없음.
API.Bible(scripture.api.bible) 무료 플랜에도 한국어 없음 확인 완료.
bible.helloao.org의 `kor_old`가 유일한 무료 한국어 성경 API.

## 현재 기능 (최신 상태)

### 1. 인트로 애니메이션 (3초)
- 페이지 로드 시 전체 화면 오버레이
- 십자가 + 확장 링 + 글로우 효과 + "오늘의 말씀" 텍스트
- 3초간 재생 후 페이드아웃, 동시에 API 호출 병렬 실행

### 2. 랜덤 말씀 표시
- 4개 카테고리의 전체 구절(ALL_VERSES)에서 랜덤 1개 선택
- 인트로 완료 후 카드에 말씀 텍스트 + 장절 레퍼런스 표시
- 카드 등장 시 reveal 애니메이션 (fade + scale)

### 3. 폭죽/컨페티 효과
- 말씀 카드 등장 시 스파클(✨) + 컨페티 조각 40개 애니메이션
- 3.5~4초 후 DOM에서 자동 제거

### 4. 클립보드 복사
- 복사 버튼 클릭 시 `말씀 텍스트\n- 장절` 형식으로 복사
- 복사 완료 시 "복사됨" 피드백 (2초), 색상 변경(녹색)

## 큐레이션된 구절 목록

4개 카테고리, 총 78구절이 `VERSES` 객체에 하드코딩:

| 카테고리 | 키 | 구절 수 | 대표 구절 |
|---|---|---|---|
| 위로가 필요할 때 | `comfort` | 21 | 시편 23:4, 이사야 41:10, 마태복음 11:28 |
| 소망/사랑/믿음 | `faith` | 20 | 요한복음 3:16, 고전 13:13, 히브리서 11:1 |
| 지혜/용기 | `wisdom` | 19 | 여호수아 1:9, 잠언 1:7, 야고보서 1:5 |
| 감사/찬양 | `praise` | 18 | 시편 100:4, 시편 150:6, 살전 5:18 |

### 구절 데이터 형식
```js
{ b: "PSA", n: "시편", c: 23, v: 4 }
// b: 책 API ID, n: 한글 책 이름, c: 장, v: 절
```

## UX 흐름

1. 페이지 로드 → 인트로 애니메이션 시작 + API 호출 동시 실행
2. 3초 대기 (인트로 완료 + API 응답 대기)
3. 인트로 페이드아웃 → 카드 reveal + 폭죽 효과
4. 말씀 텍스트 + 장절 + 복사 버튼 표시
5. 새 말씀을 보려면 페이지 새로고침

## 디자인

- **배경**: 어두운 그라데이션 (`#0f0c29` → `#302b63` → `#24243e`)
- **카드**: 글래스모피즘 (반투명 + backdrop-filter blur)
- **폰트**: Noto Serif KR (본문), sans-serif (UI)
- **강조색**: `#a8a4f0` (라벤더 퍼플)

## 과거 의사결정 기록

1. **API 선택**: API.Bible 무료 플랜에 한국어 없음 확인 → bible.helloao.org 채택
2. **번역본**: 개역개정/새번역은 저작권 문제 → 개역한글(1961, 퍼블릭도메인) 사용
3. **카테고리 버튼**: 초기에 4개 카테고리 선택 버튼 존재 → 이후 제거, 전체 랜덤으로 변경
4. **"새로운 말씀 보기" 버튼**: 초기에 존재 → 제거됨 (새로고침으로 대체)

## Git 참고

- 원격 URL에 PAT 토큰이 포함되어 있을 수 있음
- push 시: `git push origin main`
