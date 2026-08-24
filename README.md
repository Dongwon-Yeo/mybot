# 📄 Daily Insights — GitHub Pages 자동화 예제

매일 GitHub Actions가 자동으로 데이터를 수집해서 페이지를 업데이트하는 학습용 프로젝트입니다.

---

## 📁 파일 구조

```
claudeclonetest/
├─ index.html                          # 페이지 뼈대 (구조) + FAQ 챗봇 UI
├─ style.css                           # 디자인
├─ script.js                           # 기능 (JSON fetch & 렌더링)
├─ chatbot.js                          # FAQ 사이드 챗봇 (키워드 검색)
├─ requirements.txt                    # 파이썬 라이브러리 목록
├─ docs/                               # ChatGPT · Claude Code 가이드 PDF
├─ data/
│  ├─ weather.json                     # 날씨 + 코디 (Open-Meteo API)
│  ├─ news.json                        # 네이버뉴스 스크래핑 결과
│  ├─ finance.json                     # 시장 지수 (yfinance)
│  ├─ bizinfo.json                     # 기업마당 지원사업
│  └─ faq.json                         # 챗봇 Q&A (docs PDF 기반)
├─ scripts/
│  └─ collector.py                     # 데이터 수집 스크립트 (한 파일)
└─ .github/workflows/
   ├─ daily-collect.yml                # 매일 KST 06:00 전체 수집
   └─ supports-collect.yml            # 6시간마다 지원사업만 수집
```

---

## 🚀 GitHub Pages 배포 방법

1. 이 파일들을 저장소에 업로드합니다
2. **Settings → Pages → Source: Deploy from a branch → main** 설정
3. 1~2분 후 `https://jk0601.github.io/claudeclonetest` 접속

---

## 🤖 GitHub Actions 자동 실행 시각

| 워크플로우 | 실행 주기 | UTC cron |
|-----------|---------|---------|
| `daily-collect.yml` | 매일 오전 6시 (KST) | `0 21 * * *` |
| `supports-collect.yml` | 6시간마다 | `0 */6 * * *` |

---

## 📊 수집 방식 요약

| 데이터 | 방식 | 출처 |
|--------|------|------|
| 날씨·코디 | Open-Meteo 공개 API (키 불필요) | api.open-meteo.com |
| 뉴스 | requests + BeautifulSoup 스크래핑 | news.naver.com/section/101 |
| 시장 지수 | yfinance 라이브러리 | Yahoo Finance |
| 지원사업 | requests + BeautifulSoup 스크래핑 | bizinfo.go.kr |

---

## 🎓 학습 핵심 흐름

```
collector.py 실행
  ├─ Open-Meteo API  →  data/weather.json
  ├─ 네이버뉴스 스크래핑  →  data/news.json
  ├─ yfinance  →  data/finance.json
  └─ 기업마당 스크래핑  →  data/bizinfo.json
       ↓
GitHub Actions가 cron으로 자동 실행 & commit & push
       ↓
GitHub Pages가 JSON을 fetch()해서 화면에 카드로 표시
```

| 파일 | 배울 수 있는 것 |
|------|----------------|
| `index.html` | HTML 구조, 시맨틱 태그 |
| `style.css` | CSS 변수, Grid/Flexbox, 반응형 |
| `script.js` | fetch API, async/await, DOM 조작 |
| `collector.py` | requests, BeautifulSoup, yfinance, JSON 저장 |
| `daily-collect.yml` | GitHub Actions cron 자동화 |
| `chatbot.js` + `data/faq.json` | 우하단 FAQ 사이드 챗봇 (API 키 없음) |
