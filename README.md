# Daily Insights — 일일 인사이트 웹앱

> **GitHub Pages 무료 배포 | API 키·Secret 전혀 없음 | 모바일 반응형**
>
> 오늘의 영어 명언 · 날씨·코디 · 경제 뉴스 · 시장 지수 · 서울·경기 지원사업 · FAQ 챗봇을 한 페이지에서!

---

## 목차

1. [프로젝트 소개](#1-프로젝트-소개)
2. [기능 설명](#2-기능-설명)
3. [폴더 구조](#3-폴더-구조)
4. [로컬에서 바로 열기](#4-로컬에서-바로-열기)
5. [GitHub 저장소 만들기 & 업로드](#5-github-저장소-만들기--업로드)
6. [GitHub Pages 활성화](#6-github-pages-활성화)
7. [GitHub Actions로 데이터 자동 갱신](#7-github-actions로-데이터-자동-갱신)
8. [FAQ 챗봇 업데이트 방법](#8-faq-챗봇-업데이트-방법)
9. [자주 발생하는 문제 해결](#9-자주-발생하는-문제-해결)
10. [파일별 역할 요약](#10-파일별-역할-요약)

---

## 1. 프로젝트 소개

### 이 프로젝트가 특별한 이유

| 항목 | 내용 |
|------|------|
| **비용** | 완전 무료 (GitHub Pages 호스팅) |
| **API 키** | 전혀 없음 — Open-Meteo + 웹 스크래핑 + yfinance + 정적 FAQ |
| **백엔드** | 없음 — GitHub Actions가 Python으로 데이터 수집 후 JSON으로 저장 |
| **기술 스택** | HTML + CSS + Vanilla JS (빌드 도구 없음) |
| **모바일** | 완전 반응형 (모바일·태블릿·데스크톱) |
| **배포 방식** | Deploy from a branch (`main` 루트) |

### 아키텍처 원리

```
GitHub Actions
  ├─ daily-collect.yml     (매일 06:00 KST)
  │     └─ collector.py
  │           ├─ Open-Meteo     → data/weather.json
  │           ├─ 네이버 뉴스    → data/news.json
  │           ├─ yfinance       → data/finance.json
  │           └─ 기업마당       → data/bizinfo.json
  │
  └─ supports-collect.yml  (6시간마다)
        └─ 지원사업만 갱신     → data/bizinfo.json
                                       │
                              JSON 커밋 & 푸시
                                       │
                    GitHub Pages (branch 배포) 자동 반영
                                       │
                        브라우저에서 JSON만 fetch해서 렌더링
```

**핵심 포인트:** 프론트엔드(브라우저)는 `data/*.json`만 읽습니다.  
스크래핑·API 호출은 전부 Actions(서버 측)에서 합니다.

---

## 2. 기능 설명

### ① 영어 명언 (Hero 섹션)

- `script.js` 안에 명언 목록이 들어 있습니다.
- 페이지 로드 시 · `↻ 다른 명언 보기` 클릭 시 랜덤으로 바뀝니다.
- 영어 원문 → 한글 해석 → 작자 순으로 표시됩니다.

### ② 오늘의 날씨 + 코디 추천

- **날씨 소스:** Open-Meteo 공개 API (키 불필요) — 서울 기준
- **공기질:** Open-Meteo Air Quality (PM10)
- **갱신 주기:** 매일 06:00 KST (`daily-collect.yml`)
- 최고/최저 기온·강수·날씨 코드를 반영해 코디 문구·이미지를 생성합니다.

### ③ 오늘의 주요 뉴스

- **소스:** 네이버 뉴스 경제 섹션 스크래핑
- 제목 클릭 시 새 창으로 원문 이동
- **갱신 주기:** 매일 06:00 KST

### ④ 경제 / 시장 체크

- **소스:** yfinance
- **지수:** KOSPI, S&P 500, 반도체(SOX), 금(Gold)
- 전일 대비 등락·등락률 표시
- **갱신 주기:** 매일 06:00 KST

### ⑤ 서울·경기 지원사업

- **소스:** 기업마당(bizinfo.go.kr) 스크래핑
- 공고 제목 · 기간 · 상세 링크
- **갱신 주기:** 6시간마다 (`supports-collect.yml`) + 매일 전체 수집에도 포함

### ⑥ FAQ 사이드 챗봇

- **우하단 플로팅 버튼** 클릭 → 사이드 패널 슬라이드 인
- **키워드 검색 방식** (런타임 AI API 호출 없음, 키 불필요)
  - `q`·`tags` 매칭 점수가 높으면 명확한 답변 표시
  - 약한 매칭이면 "관련 항목" 버튼 제시
- **FAQ 내용:** ChatGPT · Claude Code 사용 가이드 (`docs/` PDF 기반, 약 50개+ Q&A)
- 데이터 파일: `data/faq.json` · 로직: `chatbot.js`

---

## 3. 폴더 구조

```
aishbot/   (또는 저장소 이름)
├── index.html                 ← 페이지 구조 + FAQ 챗봇 UI
├── style.css                  ← 디자인 (포레스트 그린 톤, 반응형)
├── script.js                  ← 명언 · 날씨 · 뉴스 · 금융 · 지원사업 렌더링
├── chatbot.js                 ← FAQ 사이드 챗봇 (키워드 검색)
├── requirements.txt           ← Python 라이브러리 목록
├── README.md
│
├── docs/                      ← 원본 가이드 PDF
│   ├── ChatGPT_사용가이드.pdf
│   └── 클로드코드_사용가이드.pdf
│
├── data/                      ← Actions가 갱신하거나 정적 FAQ
│   ├── weather.json           ← 날씨 + 코디
│   ├── news.json              ← 네이버 뉴스
│   ├── finance.json           ← 시장 지수
│   ├── bizinfo.json           ← 서울·경기 지원사업
│   ├── faq.json               ← 챗봇 Q&A
│   └── contents.json          ← (참고용/이전 샘플, 메인 화면은 위 JSON 사용)
│
├── scripts/
│   └── collector.py           ← 데이터 수집 스크립트 (한 파일)
│
└── .github/workflows/
    ├── daily-collect.yml      ← 매일 KST 06:00 전체 수집
    ├── supports-collect.yml   ← 6시간마다 지원사업만 수집
    └── clean-history.yml      ← 오래된 Actions 실행 기록 정리
```

---

## 4. 로컬에서 바로 열기

### 방법 1: VS Code / Cursor Live Server

1. `jkai-ai.github.io`(또는 `aishbot`) 폴더를 엽니다.
2. `index.html`을 엽니다.
3. **"Go Live"** 등으로 로컬 서버를 켭니다.

### 방법 2: Python 내장 서버

```bash
cd jkai-ai.github.io
python -m http.server 8000
```

브라우저에서 `http://localhost:8000` 접속

### 방법 3: Node.js serve

```bash
npm install -g serve
serve .
```

> **주의:** `index.html`을 더블클릭해서 `file://`로 열면  
> `fetch()`가 차단되어 JSON·FAQ를 불러오지 못합니다.  
> 반드시 `http://localhost:...` 로 여세요.

### 로컬에서 수집 스크립트만 실행

```bash
pip install -r requirements.txt
python scripts/collector.py
```

---

## 5. GitHub 저장소 만들기 & 업로드

### Step 1: GitHub 저장소 생성

1. [github.com](https://github.com) 로그인
2. 우상단 `+` → **New repository**
3. Repository name 예: `aishbot`
4. **Public** 선택 (Pages 무료 배포는 Public 권장)
5. **Create repository**

### Step 2: 로컬에서 업로드

```bash
cd jkai-ai.github.io

git init
git add .
git commit -m "feat: Daily Insights 초기 구성"

# YOUR_GITHUB_USERNAME / REPO_NAME 을 본인 정보로 교체
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/REPO_NAME.git
git branch -M main
git push -u origin main
```

### 두 계정(저장소)에 같이 올리고 싶을 때

```bash
# 예: jk0601 + aishcokr
git remote add origin https://github.com/jk0601/aishbot.git
git remote add aishco  https://github.com/aishcokr/aishbot.git

git push origin main
git push aishco main
```

> `git pull` / `git push` 기본 대상은 **추적 중인 remote**(`origin` 등)입니다.  
> Active GitHub 계정(`gh auth status`)은 **인증(누구 권한으로)** 이고, remote URL은 **어디로** 보낼지입니다.

---

## 6. GitHub Pages 활성화

이 프로젝트는 **Deploy from a branch** 방식을 사용합니다.  
(`my-ai-assistant`처럼 `deploy.yml` + Source: GitHub Actions 가 아닙니다.)

1. 저장소 → **Settings** → **Pages**
2. **Source:** Deploy from a branch
3. **Branch:** `main` / folder: `/ (root)` → Save
4. 1~2분 후 Actions에 `pages-build-deployment`가 나타나면 배포 중
5. 주소 예: `https://YOUR_GITHUB_USERNAME.github.io/REPO_NAME/`

> **배포 확인:** Settings → Pages 상단 URL, 또는 Actions의 `pages-build-deployment` 성공(✓)

### Actions 배포 vs Branch 배포 (참고)

| | Deploy from a branch | GitHub Actions (`deploy.yml`) |
|--|----------------------|-------------------------------|
| 설정 | 브랜치만 선택 | 워크플로 + Source를 Actions로 |
| 이 프로젝트 | **사용 중** | 없음 (정적 HTML이라 불필요) |
| URL 표시 | `pages-build-deployment` 실행 기록 | deploy 워크플로 environment URL |

---

## 7. GitHub Actions로 데이터 자동 갱신

### 실행 시각

| 워크플로 | 주기 | cron (UTC) |
|----------|------|------------|
| `daily-collect.yml` | 매일 오전 6시 (KST) | `0 21 * * *` |
| `supports-collect.yml` | 6시간마다 | `0 */6 * * *` |
| `clean-history.yml` | 매주 일요일 09:00 (KST) + 수동 | `0 0 * * 0` |

### 자동 실행 흐름

```
cron / 수동 실행
     │
     ├── collector.py (또는 지원사업만)
     │      → data/*.json 갱신
     │
     ├── git commit & push
     │
     └── Pages(branch)가 최신 JSON 반영
```

### 수동 실행 (데이터 바로 채우기)

1. 저장소 → **Actions**
2. 왼쪽 **Daily Data Collection** 또는 **Supports Data Collection (6h)**
3. **Run workflow** → **Run workflow**
4. 1~2분 후 완료 → `data/*.json` 커밋 확인

### Secret 설정

**없음.** Open-Meteo · 스크래핑 · yfinance 모두 무인증입니다.

---

## 8. FAQ 챗봇 업데이트 방법

챗봇은 `data/faq.json`을 검색합니다.

### FAQ 항목 추가/수정

```json
{
  "q": "질문 내용을 자연스럽게 적으세요",
  "a": "답변 내용. 여러 줄은 \\n으로 구분합니다.\n줄바꿈 예시입니다.",
  "tags": ["키워드1", "키워드2", "검색될단어"]
}
```

**태그 팁**
- 사용자가 칠 법한 단어·동의어를 넣습니다. (예: `클로드코드`, `Claude Code`, `Claude`)
- 짧고 핵심적인 단어 위주

### PDF가 바뀌었을 때

1. 새 PDF를 `docs/`에 넣습니다.
2. `data/faq.json`을 직접 수정하거나, ChatGPT/Claude에 PDF 내용을 주고 FAQ JSON을 생성해 달라고 요청합니다.

```
프롬프트 예시:
아래 내용을 바탕으로 FAQ를 만들어줘.
형식: [{ "q": "...", "a": "...", "tags": ["...", "..."] }]
질문 20개 이상, 초보자도 이해할 수 있게 답변해줘.
```

---

## 9. 자주 발생하는 문제 해결

### 날씨/뉴스/시세가 "불러오는 중"에서 멈춤

**원인:** `data/*.json` 없음 · 형식 오류 · 로컬을 `file://`로 염  
**해결:**
1. Actions에서 **Daily Data Collection** 수동 실행
2. GitHub에서 `data/weather.json` 등 확인
3. 로컬은 `python -m http.server`로 접속

### 스크래핑 실패로 Actions가 빨간색(✗)

**원인:** 네이버·기업마당 HTML 구조 변경  
**해결:**
1. `scripts/collector.py`의 셀렉터 확인
2. 브라우저 F12로 현재 구조에 맞게 수정 후 push

### 챗봇이 FAQ를 못 불러옴

**원인:** `data/faq.json` 없음 또는 JSON 문법 오류  
**해결:** 파일 존재 확인 · [jsonlint.com](https://jsonlint.com)으로 검증

### Actions에 배포 URL 워크플로가 안 보임

**원인:** Pages를 아직 안 켰거나, 이 저장소에 `deploy.yml`이 없음  
**해결:** Settings → Pages → **Deploy from a branch** 설정  
→ `pages-build-deployment`가 생겨야 정상

### GitHub Pages 404

**원인:** Pages 미설정 · 배포 진행 중 · 저장소/경로 이름 불일치  
**해결:**
1. Settings → Pages에서 `main` / `(root)` 확인
2. Actions의 `pages-build-deployment` 완료 여부 확인 (1~5분)

### 로컬이 원격보다 뒤처져 push가 거절됨

**원인:** Actions가 JSON을 커밋해 원격이 앞섬  
**해결:**
```bash
git pull origin main
git push origin main
```
`index.html`만 수정했다면 `git add index.html`로 그 파일만 커밋하면 됩니다.  
pull로 들어온 JSON은 Actions 커밋이지, 내 커밋에 억지로 넣을 필요는 없습니다.

---

## 10. 파일별 역할 요약

| 파일 | 역할 | 수정 빈도 |
|------|------|-----------|
| `index.html` | 페이지 구조 + 챗봇 UI | UI 변경 시 |
| `style.css` | 전체 스타일 · 챗봇 · 반응형 | 디자인 변경 시 |
| `script.js` | 명언 · 각 섹션 JSON 렌더링 | 거의 없음 |
| `chatbot.js` | FAQ 검색 · 패널 열기/닫기 | 거의 없음 |
| `data/weather.json` | 날씨·코디 (Actions) | 자동 |
| `data/news.json` | 뉴스 (Actions) | 자동 |
| `data/finance.json` | 시장 지수 (Actions) | 자동 |
| `data/bizinfo.json` | 지원사업 (Actions) | 자동 |
| `data/faq.json` | 챗봇 Q&A | 가이드 변경 시 |
| `docs/*.pdf` | FAQ 원본 자료 | 자료 교체 시 |
| `scripts/collector.py` | 데이터 수집 | 셀렉터/소스 변경 시 |
| `daily-collect.yml` | 매일 전체 수집 | 거의 없음 |
| `supports-collect.yml` | 6시간 지원사업 수집 | 거의 없음 |
| `clean-history.yml` | Actions 실행 기록 정리 | 거의 없음 |

---

## 기술 스택 & 라이선스

- **Frontend:** HTML5 · CSS3 (Custom Properties, Grid) · Vanilla JavaScript
- **Data:** Python 3 · requests · beautifulsoup4 · lxml · yfinance
- **Hosting:** GitHub Pages (Deploy from a branch)
- **CI/CD:** GitHub Actions (수집 · 기록 정리)
- **외부 데이터:** Open-Meteo · 네이버 뉴스 · Yahoo Finance(yfinance) · 기업마당

이 프로젝트는 **교육용** 자료입니다.  
뉴스·지원사업 등은 공개 페이지 스크래핑이며, 개인·소량 학습 목적입니다.  
최신 요금·기능 정보는 각 서비스 공식 문서를 확인하세요.
