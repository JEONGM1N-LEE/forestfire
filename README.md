> 수성인재교육 실무형 AI/SW 인재 육성 Lab 팀 프로젝트

# FORESTFIRE ATLAS KOREA

전국 시군구 단위의 산불 발생 가능성을 예측하고 지도 기반으로 결과를 제공하는 AI 웹서비스

산불 이력과 기상 데이터를 활용해 XGBoost 기반으로 산불 발생 가능성을 예측하고,
Next.js · Express · Flask로 구성된 웹서비스를 통해 예측 결과를 제공합니다.

화면에 표시되는 당일·시나리오 위험도 값은 XGBoost `predict_proba`의 raw 확률(`ml_risk`)을 ×100한 **산불위험지수(0~100)** 입니다.


## 목차

- [프로젝트 개요](#프로젝트-개요)
- [주요 기능](#주요-기능)
- [Tech Stack](#tech-stack)
- [시스템 아키텍처](#시스템-아키텍처)
- [My Role](#my-role)
- [머신러닝 모델](#머신러닝-모델)
- [데이터 구성](#데이터-구성)
- [주요 서비스 구조](#주요-서비스-구조)
- [주요 화면](#주요-화면)
- [폴더 구조](#폴더-구조)
- [실행 방법](#실행-방법)
- [환경변수](#환경변수)
- [Docker](#docker)
- [회원·챗봇·보고서](#회원챗봇보고서)
- [주요 API](#주요-api)
- [데이터 및 지도 운영](#데이터-및-지도-운영)
- [예측 모델 운영](#예측-모델-운영)
- [배치 파이프라인](#배치-파이프라인)
- [프로젝트를 통해 배운 점](#프로젝트를-통해-배운-점)


## 프로젝트 개요

산불은 기상 조건과 과거 산불 발생 이력 등 여러 요인의 영향을 받기 때문에
지역별 데이터를 활용한 산불 발생 가능성 예측이 필요합니다.

FORESTFIRE ATLAS KOREA는 시군구 단위의 산불 이력과 기상 데이터를 기반으로
일별 산불 발생 가능성을 예측하고, 이를 지도와 웹 화면에서 확인할 수 있도록 구현한 서비스입니다.

단순히 머신러닝 모델의 예측 결과만 제공하는 것이 아니라,
ML 모델을 별도의 Flask 서비스로 분리하고 Express와 Next.js를 통해
사용자 화면까지 연결하는 구조로 구성했습니다.


## 주요 기능

- 전국 시군구 단위 산불 위험 지도
- 일별 산불 발생 가능성 예측
- 사용자 지정 기상 조건을 활용한 시나리오 예측
- 과거 산불 발생 이력 조회
- 지역 및 산 통합 검색
- 기상 정보 조회
- Gemini 기반 산불 안내 챗봇
- Google / Kakao OAuth 및 로컬 로그인
- 지역별 PDF 리포트 생성 및 다운로드
- 산불 이력 데이터 갱신


## Tech Stack

| 영역 | 기술 | 활용 |
|---|---|---|
| Frontend | Next.js, React, TypeScript | 사용자 화면 및 API 연동 |
| Backend | Express, TypeScript | REST API, 인증/세션, 서비스 로직 |
| ML Service | Python, Flask | 머신러닝 모델 예측 API |
| Machine Learning | XGBoost | 산불 발생 가능성 예측 |
| Data | Python, Pandas | 데이터 처리 및 피처 구성 |
| Database | MariaDB | 산불·기상·회원·예측 데이터 관리 |
| AI | Gemini API | 산불 안내 챗봇 |
| PDF | Jinja2, Playwright | 지역별 PDF 리포트 생성 |
| Deployment | Docker | 프론트엔드 컨테이너 실행 |
| Development | Git, GitHub, Cursor | 버전 관리 및 개발 |


## 시스템 아키텍처

프로젝트는 Frontend, Backend, ML Service를 분리하고,
외부 데이터를 GitHub Actions를 통해 MariaDB에 적재한 뒤
ETL과 머신러닝 모델에서 활용하는 구조로 구성했습니다.

### 전체 데이터 및 서비스 흐름

```text
                         ┌──────────────────────┐
                         │    External Data     │
                         │                      │
                         │ 산불 / 기상 데이터  │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    GitHub Actions    │
                         │                      │
                         │ 데이터 수집·적재 자동화 │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │       MariaDB        │
                         │                      │
                         │ 산불 이력 / 기상 데이터 │
                         └───────┬────────┬─────┘
                                 │        │
                          학습 데이터│        │서비스 데이터
                                 │        │
                                 ▼        ▼
                    ┌────────────────┐   ┌──────────────────┐
                    │      ETL       │   │    Express       │
                    │                │   │   Backend :4000  │
                    │ 전처리·피처 구성 │   │                  │
                    └───────┬────────┘   │ API / 회원 / DB  │
                            │              │ Gemini / Proxy   │
                            ▼              └────────┬─────────┘
                    ┌────────────────┐              │
                    │  XGBoost Model │              │
                    │                │              │
                    │    모델 학습   │              │
                    └───────┬────────┘              │
                            │                       │
                            ▼                       │
                    ┌────────────────┐              │
                    │     Flask      │◄─────────────┘
                    │   ML Service   │
                    │     :5000      │
                    │                │
                    │ 모델 예측 / PDF │
                    └───────▲────────┘
                            │
                    ┌───────┴────────┐
                    │    Next.js     │
                    │    Frontend    │
                    │      :3000     │
                    └───────▲────────┘
                            │
                    ┌───────┴────────┐
                    │    Browser     │
                    └────────────────┘
```

### 데이터 및 모델 흐름

```text
External Data
     ↓
GitHub Actions
     ↓
MariaDB
     ↓
데이터 전처리
     ↓
Feature Construction
     ↓
XGBoost Model Training
     ↓
학습된 모델
     ↓
Flask ML Service
```

GitHub Actions에서는 외부 데이터를 수집하고 MariaDB에 적재합니다.
적재된 데이터는 ETL 과정에서 전처리 및 피처 구성에 활용되며,
XGBoost 모델 학습에 사용됩니다.

### 서비스 요청 흐름

```text
Browser
   ↓
Next.js :3000
   ↓
Route Handler
   ↓
Express :4000
   ↓
Flask :5000
   ↓
XGBoost Prediction
   ↓
결과 반환
   ↓
Express
   ↓
Next.js
   ↓
Browser
```

Express는 회원/세션 관리, 데이터 처리, 외부 서비스 연동 및 Flask 프록시 역할을 담당하고,
Flask는 머신러닝 모델을 이용한 예측을 담당하도록 역할을 분리했습니다.


## My Role

팀 프로젝트에서 데이터 및 머신러닝 영역을 중심으로 참여했습니다.

### 머신러닝

- 팀에서 수집·전처리한 산불 및 기상 데이터를 기반으로 피처 구성
- XGBoost 모델 학습
- 모델 평가
- 산불 발생 가능성 예측 로직 구현
- 학습된 모델의 예측 결과가 서비스에서 활용되는 흐름 확인

### 서비스 연결 및 배포

- ML Service의 예측 구조와 API 흐름 분석
- 프론트엔드에서 예측 결과가 사용자 화면까지 전달되는 전체 흐름 확인
- 프론트엔드 배포 참여
- 데이터 → 머신러닝 모델 → API → 웹 화면으로 이어지는 서비스 구조 이해

프로젝트 진행 과정에서 개별 모델 구현에 그치지 않고,
실제 웹서비스에서 머신러닝 결과가 어떻게 활용되는지 이해하는 데 중점을 두었습니다.


## 머신러닝 모델

### 예측 대상

시군구 × 일 단위의 산불 발생 가능성을 XGBoost로 예측합니다.

웹서비스에서는 XGBoost의 `predict_proba`에서 얻은 raw 확률값을
100배하여 0~100 범위의 산불위험지수로 표시합니다.

```text
XGBoost predict_proba
        ↓
    raw probability
        ↓
       × 100
        ↓
산불위험지수 (0~100)
```

### 사용 피처

현재 모델에서는 다음 10개 피처를 사용합니다.

```text
temp_avg
precip
wind_avg
humidity_avg
hist_fire_rate
hist_fire_count_365
dwi
precip_sum_7d
precip_sum_14d
dry_days
```

기상 조건뿐만 아니라 과거 산불 발생 이력과 최근 강수량,
건조 일수 등의 정보를 함께 활용하도록 구성했습니다.

### 학습 및 추론 구조

| 단계 | 위치 |
|---|---|
| 데이터 전처리 | `etl/` |
| 모델 학습 | `etl/ml/train_wildfire_xgb.py` |
| 모델 저장 | `ml-service/models/` |
| 예측 | `ml-service/predict/daily.py` |
| Flask API | `ml-service/routes/` |

학습된 XGBoost 모델은 `ml-service`에서 로드하여 예측에 사용합니다.


## 데이터 구성

### 산불 이력 데이터

MariaDB의 `forestfire_stats`를 주요 데이터 소스로 사용합니다.

```text
MariaDB
forestfire_stats
       │
       ├── 산불 이력 조회
       ├── 모델 학습
       └── 지도 데이터 갱신
```

DB 사용이 어려운 경우 일부 데이터는 CSV를 활용할 수 있도록
fallback 구조를 구성했습니다.

### 기상 데이터

기상청 ASOS API를 활용하여 당일 기상 데이터를 가져오고,
MariaDB에 저장된 시군구별 과거 기상 데이터를 활용하여
예측에 필요한 lag 및 통계 피처를 구성합니다.

주요 데이터는 다음과 같습니다.

- 평균 기온
- 강수량
- 평균 풍속
- 평균 상대습도
- 최근 강수량
- 건조 일수

### 데이터 소스

| 용도 | 우선 소스 | 비고 |
|---|---|---|
| 산불 이력 (맵 갱신·학습·분석) | MariaDB `forestfire_stats` | 실패 시 `refined_wildfire_data.csv` 폴백 |
| 예측용 당일 기상 | 기상청 ASOS API | `KMA_API_AUTH_KEY` |
| 예측용 lag 기상 (어제·그저께) | MariaDB `weather_daily_sigungu` | 실패 시 CSV 폴백 |
| 학습용 기상 | MariaDB 우선 | 동일 테이블 / CSV 폴백 |
| 지도 JSON | `frontend/public/data` + `backend/data` | Express 불가 시 frontend 데이터 폴백 |
| 챗봇·리포트용 당일 예측 스냅샷 | `backend/data/daily_ml_risk.json` | MariaDB에도 예측 결과 저장 |
| 산 썸네일 | `frontend/public/data/mountain-images/` | 오프라인 ETL에서 산림청 API 활용 |
| 지역명 정규화 | `legal-dong-lookup.json` | frontend/backend에서 각각 활용 |

`refined_wildfire_data.csv`는 더 이상 주 데이터가 아닙니다.
DB가 정상적으로 동작하면 예측·이력 갱신·학습 과정에서 해당 CSV 없이 운영할 수 있습니다.

### 데이터 흐름

```text
산불 이력 ─────┐
               │
               ▼
          데이터 처리
               │
기상 데이터 ───┤
               ▼
        Feature Construction
               │
               ▼
         XGBoost Model
               │
               ▼
       산불 발생 가능성
               │
               ▼
          Flask API
               │
               ▼
        Express Backend
               │
               ▼
          Next.js UI
```


## 주요 서비스 구조

### Frontend

`frontend/`

Next.js와 TypeScript를 기반으로 사용자 화면을 구성합니다.

주요 역할:

- 산불 위험 지도
- 지역 및 산 검색
- 산불 예측 결과 표시
- 시나리오 예측
- 기상 정보 표시
- 로그인 UI
- 챗봇 UI
- PDF 리포트 UI

Next.js Route Handler를 통해 Express Backend와 통신하도록 구성했습니다.

### Backend

`backend/`

Express와 TypeScript를 기반으로 API와 서비스 로직을 담당합니다.

주요 역할:

- REST API
- 회원가입 및 로그인
- 세션 관리
- Google / Kakao OAuth
- Flask ML Service 연동
- Gemini 챗봇 연동
- 산불 이력 데이터 갱신
- 예측 데이터 관리
- PDF 리포트 요청 처리

### ML Service

`ml-service/`

Flask 기반의 머신러닝 서비스입니다.

주요 역할:

- XGBoost 모델 로딩
- 일별 산불 예측
- 시나리오 예측
- 예측 결과 반환
- PDF 리포트 생성 지원
- 예측 결과 DB 저장

머신러닝 영역을 웹 Backend와 별도의 서비스로 분리하여
모델과 웹 애플리케이션의 역할을 구분했습니다.

### ETL

`etl/`

웹서비스 요청과 분리된 오프라인 데이터 처리 영역입니다.

주요 역할:

- 산불 데이터 전처리
- 기상 데이터 전처리
- 데이터 분석
- 지도 데이터 생성
- 산 정보 수집
- 지역명 lookup 생성
- 머신러닝 모델 학습

주요 실행 파일:

```text
etl/pipeline/preprocess.py
etl/pipeline/preprocess_weather.py
etl/pipeline/load_korea_mountains.py
etl/pipeline/build_legal_dong_lookup.py
etl/analyze/analyze_wildfire_mountain_events.py
etl/map/build_admin_layers.py
etl/map/export_map_data.py
etl/pipeline/fetch_mountain_images.py
etl/map/compress_web_data.py
etl/ml/train_wildfire_xgb.py
```


## 주요 화면

- 좌측: FORESTFIRE ATLAS KOREA 브랜드, 지역·산 통합 검색, 위험 표시(당일 예측 / 사용자 지정 / 과거 이력), 최근 산불 발생 피드
- 지도: SVG 행정구역 + 카카오 위성. 산 검색은 파란 핀, 산불 이력 클릭은 빨간 핀
- 지도 범례: 예측 모드에서 산불위험지수 0~100, 예측일, AUC 표시
- 우측 패널: 전체 산불 건수·갱신 날짜·갱신 버튼
- 지역 미선택 시 전국 평균 및 최고 위험 시도 요약
- 지역 선택 시 산불 이력 및 산 상세 정보 표시
- 로그인 시 지도 상단에서 지역별 보고서 기능 제공
- 사용자 지정 예측: 접속월부터 12개월을 대상으로 기상 조건을 조정하여 시나리오 예측
- 기상 카드: 선택 지역이 있으면 해당 시군구 또는 시도 평균 기상 정보 표시


## 런타임 구조

| 프로세스 | 폴더 | 포트 | 역할 |
|---|---|---:|---|
| Next.js | `frontend/` | 3000 | UI · `/api` → Express 프록시 · 챗봇·로그인·보고서 모달 · 산 이미지 정적 파일 |
| Express | `backend/` | 4000 | 공개 API · 회원/세션 · 챗봇 · 산불이력 맵 갱신 · 보고서 게이트 · Flask 프록시 · DTO 화이트리스트 |
| Flask | `ml-service/` | 5000 | 예측 · PDF 렌더(Playwright) · localhost 전용 · 당일 예측 DB 스냅샷 |
| 배치 ETL | `etl/` | — | 전처리·학습·산 이미지·법정동 lookup |

서비스 요청은 다음과 같이 전달됩니다.

```text
브라우저
   ↓
Next.js :3000
   ↓ Route Handler
Express :4000
   ↓
Flask :5000
```

`/api/*` 요청은 `frontend/src/app/api/[...path]/route.ts`가 Express로 전달합니다.

쿠키·`Set-Cookie`와 PDF의 `Content-Disposition`도 전달하며,
브라우저에는 Express에서 필터링한 DTO만 전달합니다.

파이썬 stdout, 모델 경로, API Key 등의 내부 정보는 응답에 포함하지 않습니다.


## 폴더 구조

```text
ForestFire/
│
├── frontend/                  Next.js UI (:3000, Docker standalone)
│   ├── public/data/            폴백 지도 JSON · legal-dong-lookup · mountain-images
│   └── src/
│       ├── app/
│       ├── components/
│       ├── lib/
│       └── app/api/[...path]/  Express 프록시
│
├── backend/                   Express 공개 API (:4000, TypeScript)
│   ├── data/                  지도·daily_ml_risk JSON 사본
│   ├── migrations/            챗봇 세션·소셜 로그인·예측 테이블 SQL
│   └── src/
│
├── ml-service/                Flask (:5000, localhost)
│   ├── models/                XGBoost 모델 JSON
│   ├── reference/             시군구 hist · 관측소 매핑 CSV
│   ├── predict/               예측 엔진 + weather_db · fire_db · risk_snapshot_db
│   ├── report/                지역별 PDF (Jinja2 + Playwright)
│   └── routes/                health · predict · report
│
├── etl/                       오프라인 ETL · 분석 · 학습 · 산 이미지
│   ├── pipeline/
│   ├── analyze/
│   ├── map/
│   └── ml/
│
├── db/                        로컬 대용량 CSV (Git 제외)
├── db-archive/                ETL·분석 원본·중간 산출물 (Git 제외)
├── .gitignore
└── README.md
```

`db/`와 `db-archive/`는 로컬에서 사용하는 대용량 데이터 및
ETL·분석 원본/중간 산출물을 관리하는 영역이며 Git에서는 제외합니다.


## 실행 방법

이 프로젝트는 Flask, Express, Next.js의 세 프로세스를 실행하는 구조입니다.

PowerShell 기준으로 터미널을 세 개 열고 **Flask → Express → Next.js** 순으로 실행하는 것을 권장합니다.

### 1. Flask ML Service

```powershell
cd ml-service

# .env에 KMA_API_AUTH_KEY · DB_* 입력
pip install -r requirements.txt

# PDF 보고서 기능 사용 시 최초 1회
playwright install chromium

python app.py
```

기본적으로 `5000` 포트를 사용합니다.

Linux 배포 시 Chromium 시스템 의존성 및 한글 폰트는
`ml-service/README.md`를 참고합니다.

### 2. Express Backend

```powershell
cd backend

# .env에 FRONTEND_ORIGIN · ML_SERVICE_URL · GEMINI_API_KEY
# DB_* · SESSION_SECRET 등 입력
npm install
npm run dev
```

기본적으로 `4000` 포트를 사용합니다.

기동 시 당일 예측을 몇 번 재시도하여 캐시에 올려 두도록 구성되어 있습니다.

### 3. Next.js Frontend

```powershell
cd frontend

# .env.local에 NEXT_PUBLIC_KAKAO_MAP_KEY · EXPRESS_URL 입력
npm install
npm run dev
```

기본적으로 `3000` 포트를 사용합니다.

브라우저:

```text
http://localhost:3000
```

Express 헬스 체크:

```text
http://localhost:4000/api/health
```


## 환경변수

실행에 필요한 API Key, DB 인증정보 및 OAuth Secret 등은 환경변수로 관리합니다.

| 위치 | 키 | 용도 |
|---|---|---|
| `ml-service/.env` | `KMA_API_AUTH_KEY` | 기상청 ASOS (당일 예측) |
| `ml-service/.env` | `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | MariaDB (산불·lag 기상·당일 예측 스냅샷) |
| `ml-service/.env` | `ML_HOST`, `ML_PORT` | Flask 바인딩 (기본 `127.0.0.1:5000`) |
| `ml-service/.env` | `FOREST_FIRE_SERVICE_KEY` | 선택 사항. 레거시 OpenAPI 스크립트용 |
| `etl/.env` 또는 루트 `.env` | `FOREST_MOUNTAIN_SERVICE_KEY` | 산림청 산정보·산 이미지 수집 |
| `backend/.env` | `PORT`, `FRONTEND_ORIGIN`, `ML_SERVICE_URL`, `PREDICT_CACHE_MS`, `DATA_DIR` | CORS · Flask URL · 예측 캐시 · 지도 데이터 폴더 |
| `backend/.env` | `GEMINI_API_KEY`, `GEMINI_MODEL` | 안내 챗봇 |
| `backend/.env` | `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | 회원·챗봇·산불이력 동기화 |
| `backend/.env` | `SESSION_SECRET`, `SESSION_IDLE_MINUTES` | 로그인 세션 쿠키 서명 · 유휴 만료 |
| `backend/.env` | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | Google OAuth |
| `backend/.env` | `KAKAO_REST_API_KEY`, `KAKAO_CLIENT_SECRET` | Kakao OAuth |
| `backend/.env` | `OAUTH_REDIRECT_BASE` | OAuth 콜백 베이스 |
| `frontend/.env.local` | `NEXT_PUBLIC_KAKAO_MAP_KEY`, `EXPRESS_URL` | Kakao JS 키 · Express 주소 |

Kakao 개발자 콘솔에서는 **JavaScript 키**를 사용하고,
Web 플랫폼에 다음 주소를 등록해야 위성 지도가 표시됩니다.

```text
http://localhost:3000
```

Google / Kakao 로그인을 사용하는 경우 OAuth Redirect URI를 등록합니다.

Google:

```text
http://localhost:3000/api/auth/google/callback
```

Kakao:

```text
http://localhost:3000/api/auth/kakao/callback
```

다음과 같은 민감한 환경변수는 `frontend/.env.local`에 두지 않습니다.

```text
KMA_API_AUTH_KEY
DB_*
FOREST_FIRE_SERVICE_KEY
FOREST_MOUNTAIN_SERVICE_KEY
GEMINI_API_KEY
SESSION_SECRET
GOOGLE_CLIENT_SECRET
KAKAO_*
```

실제 인증정보 및 API Key는 Repository에 포함하지 않습니다.


## Docker

Frontend는 Next.js의 standalone output을 활용하여 Docker 환경에서도 실행할 수 있습니다.

```powershell
cd frontend

docker compose --env-file .env.local up -d --build
```

Frontend 컨테이너는 `3000` 포트를 사용하며,
Express와 Flask는 별도의 프로세스로 실행합니다.

`next.config.ts`는 `output: "standalone"`으로 구성되어 있습니다.

Lightsail 등에서 UI만 컨테이너로 실행하는 경우:

```powershell
cd frontend

docker compose --env-file .env.local up -d --build
```

빌드 인자로 `NEXT_PUBLIC_KAKAO_MAP_KEY`와 `EXPRESS_URL`이 들어가며,
호스트 80 → 컨테이너 3000으로 연결할 수 있습니다.


## 회원·챗봇·보고서

### 회원 기능

| 기능 | 비회원 | 회원 |
|---|---|---|
| 지도 · 당일/시나리오 예측 · 챗봇 Q&A | 가능 | 가능 |
| 챗봇 이전 대화내역 불러오기 | 불가 | 가능 |
| PDF 보고서 생성·다운로드 | 불가 | 가능 |
| 챗봇 대화 DB 저장 | 게스트 세션 | `user_id` 기준 |

회원/비회원은 로그인 세션 유무로 구분합니다.
구독 및 결제 테이블은 사용하지 않습니다.

로그인은 다음 방식을 지원합니다.

- 로컬 아이디/비밀번호
- Google OAuth
- Kakao OAuth

### 회원가입 규칙

- 아이디: 영문 소문자로 시작, 소문자+숫자만 사용, 4~20자
- 공백 및 특수문자 사용 불가
- 비밀번호: 8~20자
- 영문 대문자/소문자/숫자/특수문자 중 2가지 이상 조합
- 비밀번호 확인 일치 필수

소셜 로그인은 로그인 모달과 회원가입 모달의 동작을 구분합니다.

- 로그인 모달: 기존 소셜 계정만 허용
- 회원가입 모달: 신규 소셜 계정 생성 가능
- 신규 소셜 계정은 비밀번호 없이 생성되며 닉네임을 자동 배정
- `intent=login|register`로 동작 구분

### 유휴 세션

기본 유휴 시간은 30분입니다.

`SESSION_IDLE_MINUTES` 환경변수로 변경할 수 있습니다.

- 마지막 사용자 조작 기준 30분
- 자동 폴링은 세션 시간을 연장하지 않음
- 만료 5분 전 안내 모달 표시
- 로그아웃 또는 시간 연장 선택 가능
- 만료 시 자동 로그아웃

### AI 챗봇

Gemini API를 활용하여 산불 예측 결과와 지역 정보를 기반으로
사용자의 질문에 답변하는 안내 챗봇을 구현했습니다.

챗봇은 Express Backend에서 처리하며,
당일 예측 API 결과를 우선 사용하고 필요한 경우 저장된 예측 데이터를 fallback으로 활용합니다.

로그인 회원은 이전 대화 내역을 저장하고 다시 불러올 수 있습니다.

게스트 세션은 브라우저 `localStorage`의 `sessionId`를 사용합니다.

챗봇의 주요 동작:

- 로그인 회원은 `user_id` 기준으로 대화 이력 관리
- 게스트는 `sessionId` 기준으로 대화 관리
- 로그인 직후 게스트로 사용하던 세션은 계정 히스토리에 병합
- 이전 대화내역 불러오기 기능은 로그인 회원에게만 제공
- 챗봇 헤더에서 닉네임 및 회원/게스트 상태 표시
- 챗봇 창은 헤더 드래그를 통해 위치 이동 가능
- 플로팅 버튼은 우측 하단에 고정

### PDF 리포트

로그인한 사용자는 산불 예측 결과를 기반으로 지역별 PDF 리포트를 생성할 수 있습니다.

```text
사용자
  ↓
Express
  ↓
회원 세션 확인
  ↓
Flask ML Service
  ↓
Jinja2 + Playwright
  ↓
A4 PDF 생성
  ↓
다운로드
```

PDF는 DB에 장기간 저장하지 않고,
생성 후 메모리에 임시 보관하며 30분 TTL 동안 다운로드할 수 있도록 구성했습니다.

리포트 구성:

- 표지: 발행일 · 작성자 · 닉네임 · 요약 · 게이지
- 전국 리포트: 시군구 순위 상위 10 · 하위 5
- 특정 지역 리포트: 해당 지역 범위 유지

챗봇에서 PDF를 요청하는 경우 로그인한 사용자만 사용할 수 있으며,
대화에서 확실하게 인식된 지역명을 기준으로 리포트 지역을 결정합니다.


## 주요 API

### Express API

#### 맵 · 예측 · 동기화

```text
GET  /api/health
GET  /api/map/data
GET  /api/map/admin/:level

POST /api/predict/daily
GET  /api/predict/scenario/baseline
POST /api/predict/scenario

POST /api/wildfires/sync
GET  /api/wildfires/sync/status
```

`/api/map/admin/:level`의 `level`은 다음 값을 사용합니다.

```text
sido
sigungu
emd
```

일별 예측:

```text
POST /api/predict/daily
```

요청 예시:

```text
{ source, force, date?, weather? }
```

시나리오 baseline:

```text
GET /api/predict/scenario/baseline?month=9
```

선택한 월의 평년 및 프리셋 기상을 제공합니다.

시나리오 예측:

```text
POST /api/predict/scenario
```

요청 형식:

```text
{
  year,
  month,
  weather: {
    temp_avg,
    humidity_avg,
    wind_avg,
    precip
  }
}
```

### 회원

```text
POST /api/auth/register
POST /api/auth/login
POST /api/auth/extend
POST /api/auth/logout

GET  /api/auth/me

GET  /api/auth/google
GET  /api/auth/kakao
GET  /api/auth/google/callback
GET  /api/auth/kakao/callback
```

소셜 인증에서는 `intent=login|register`를 사용하여
로그인과 회원가입 동작을 구분합니다.

### 챗봇 · 리포트

```text
POST /api/chat
GET  /api/chat/history

GET  /api/report/daily
POST /api/report/pdf
GET  /api/report/download/:id
```

`POST /api/chat`:

```text
{
  message,
  sessionId?
}
```

`POST /api/report/pdf`:

```text
{
  regionQuery?
}
```

지역을 비워두면 전국 리포트를 생성합니다.

PDF 다운로드는 메모리 TTL 30분 동안만 유효합니다.

### Flask API

Flask API는 Express Backend에서 호출하는 내부 서비스로 사용합니다.

```text
GET  /health

POST /predict/daily
GET  /predict/scenario/baseline?month=
POST /predict/scenario

POST /report/pdf
```

Flask API는 Express를 통해서만 호출하는 내부 서비스입니다.


## 데이터 및 지도 운영

### backend와 frontend가 지도 데이터를 나눠 갖는 이유

`frontend`와 `backend`는 지도 JSON을 각각 별도로 보유합니다.

```text
backend/data
```

- 브라우저 첫 로딩(`/api/map/*`)
- 웹 산불 이력 갱신
- 챗봇 폴백

```text
frontend/public/data
```

- Express가 동작하지 않을 때의 폴백
- ETL 원본 배포본
- 산 이미지
- 법정동 lookup

원본 지도 JSON은 `etl`에서 생성합니다.

오프라인에서 `frontend/public/data`를 갱신할 때
`etl/paths.py`의 `sync_backend_data()`가 `backend/data`에도 데이터를 복사합니다.

### 산불 이력 갱신

웹에서는 우측 패널 헤더의 새로고침 버튼을 통해 산불 이력을 갱신할 수 있습니다.

API:

```text
POST /api/wildfires/sync
```

Express가 MariaDB의 `forestfire_stats`를 읽어
`backend/data`의 산불 건수 및 지도 색상 데이터를 갱신합니다.

웹 첫 화면과 새로고침 시에도 `/api/map/*`를 읽기 때문에
직전에 동기화한 데이터가 유지됩니다.

오프라인에서 `frontend/public/data`까지 갱신하려면:

```powershell
python etl/pipeline/sync_wildfire_history.py
```

기존 공공데이터 OpenAPI 증분 스크립트:

```text
etl/pipeline/sync_wildfire_openapi.py
```

해당 스크립트는 레거시 경로로 남아 있으며,
웹 버튼과 기본 동기화 경로는 MariaDB를 사용합니다.

### 당일 예측 스냅샷

당일 예측 결과는 다음 경로에 저장됩니다.

```text
backend/data/daily_ml_risk.json
```

Express가 예측 API 성공 시 해당 파일에 저장하며,
Flask는 같은 결과를 MariaDB의 다음 테이블에도 적재합니다.

```text
daily_ml_risk_runs
daily_ml_risk_regions
```

챗봇은 예측 API를 우선 사용하고,
API 호출이 실패하면 저장된 `daily_ml_risk.json`을 fallback으로 활용합니다.

### 산 이미지

산 이미지 파일은 다음 경로에 저장됩니다.

```text
frontend/public/data/mountain-images/
```

오프라인 ETL에서 산림청 산정보 OpenAPI를 활용하여 수집합니다.

```text
etl/pipeline/fetch_mountain_images.py
```

런타임에서 외부 산림청 API를 직접 호출하지 않습니다.

Git에는 `.gitkeep`만 두고 실제 이미지 파일은 제외합니다.


## 예측 모델 운영

### 모델 요약

시군구 × 일 단위 산불 발생 확률을 XGBoost로 예측합니다.

웹에서는 raw 확률(`ml_risk`)에 100을 곱하여
산불위험지수(0~100)로 표시합니다.

### 피처

```text
temp_avg
precip
wind_avg
humidity_avg
hist_fire_rate
hist_fire_count_365
dwi
precip_sum_7d
precip_sum_14d
dry_days
```

### 주요 파일

| 단계 | 위치 |
|---|---|
| 학습 | `etl/ml/train_wildfire_xgb.py` |
| 추론 | `ml-service/predict/daily.py` |
| 모델 산출물 | `ml-service/models/wildfire_xgb_*.json` |
| 시군구 이력 상태 | `ml-service/reference/sigungu_hist_state.csv` |
| DB 스냅샷 | `daily_ml_risk_runs`, `daily_ml_risk_regions` |
| DB 마이그레이션 | `backend/migrations/002_daily_ml_risk.sql` |

### CLI 예측

```powershell
cd ml-service

python -m predict.daily --kma
```

당일 KMA 예측이 완료되면 Flask가 MariaDB에도 UPSERT합니다.

시나리오 예측은 DB에 저장하지 않습니다.

테이블이 없거나 `DB_*` 환경변수가 비어 있는 경우에도
로그만 남기고 예측 HTTP 응답 자체는 유지하도록 구성했습니다.


## 배치 파이프라인

웹서비스 요청과 분리된 오프라인 ETL 및 분석·학습 작업입니다.

자세한 설명은 `etl/README.md`를 참고합니다.

주요 실행 순서는 다음과 같습니다.

```powershell
python etl/pipeline/preprocess.py
python etl/pipeline/preprocess_weather.py
python etl/pipeline/load_korea_mountains.py
python etl/pipeline/build_legal_dong_lookup.py

python etl/analyze/analyze_wildfire_mountain_events.py

python etl/map/build_admin_layers.py
python etl/map/export_map_data.py

python etl/pipeline/fetch_mountain_images.py
python etl/map/compress_web_data.py

python etl/ml/train_wildfire_xgb.py

cd ml-service
python -m predict.daily --kma
```

경로 상수는 다음 파일에서 관리합니다.

```text
etl/paths.py
```

산불 원본 로드는 다음 파일을 공통으로 사용합니다.

```text
etl/pipeline/load_wildfire_history.py
```

DB를 우선 사용하고 필요한 경우 CSV fallback 구조를 활용합니다.


## 프로젝트를 통해 배운 점

이번 프로젝트에서는 머신러닝 모델을 만드는 것뿐만 아니라
모델의 결과가 실제 웹서비스의 기능으로 연결되는 전체 흐름을 경험했습니다.

특히 데이터와 모델을 구현한 뒤에는

```text
Data
  ↓
Feature
  ↓
Model
  ↓
ML Service
  ↓
Backend API
  ↓
Frontend
  ↓
User
```

와 같이 각 영역이 연결되어야 실제 서비스가 완성된다는 점을 배웠습니다.

또한 프로젝트 진행 과정에서 AI 개발 도구를 활용하면서
생성된 코드를 그대로 사용하는 것보다
API 호출 구조와 데이터 흐름을 직접 확인하고
코드의 역할을 이해하는 것이 중요하다는 점을 경험했습니다.

세부적인 API 및 각 서비스의 개발 문서는 다음 README를 참고할 수 있습니다.

```text
backend/README.md
frontend/README.md
ml-service/README.md
etl/README.md
```
