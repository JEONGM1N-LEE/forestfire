> 수성인재교육 실무형 AI/SW 인재 육성 Lab 팀 프로젝트
# FORESTFIRE ATLAS KOREA

전국 시군구 단위의 산불 발생 가능성을 예측하고 지도 기반으로 결과를 제공하는 AI 웹서비스

산불 이력과 기상 데이터를 활용해 XGBoost 기반으로 산불 발생 가능성을 예측하고,
Next.js · Express · Flask로 구성된 웹서비스를 통해 예측 결과를 사용자에게 제공합니다.
화면에 보이는 당일·시나리오 위험도 값은 XGBoost `predict_proba` raw 확률(`ml_risk`)을 ×100 한 **산불위험지수 (0~100)** 입니다.

## 프로젝트 개요

산불은 기상 조건과 과거 산불 발생 이력 등 여러 요인의 영향을 받기 때문에
지역별 데이터를 활용한 산불 발생 가능성 예측이 필요합니다.

FORESTFIRE ATLAS KOREA는 시군구 단위의 산불 이력과 기상 데이터를 기반으로
일별 산불 발생 가능성을 예측하고, 이를 지도와 웹 화면에서 확인할 수 있도록 구현한 서비스입니다.

단순히 머신러닝 모델의 예측 결과를 제공하는 것에 그치지 않고,
ML 모델을 별도의 Flask 서비스로 분리하고 Express와 Next.js를 통해
사용자 화면까지 연결하는 구조로 구성했습니다.


## 주요 기능

- 전국 시군구 단위 산불 위험 지도
- 일별(실시간) 산불 발생 가능성 예측
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
GitHub Actions를 활용한 데이터 적재 및 모델 운영 흐름을 구성했습니다.

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
                    │  Flask         │◄─────────────┘
                    │  ML Service    │
                    │   :5000        │
                    │                │
                    │ 모델 예측 / PDF │
                    └────────────────┘
                            ▲
                            │
                    ┌───────┴────────┐
                    │    Next.js     │
                    │   Frontend     │
                    │     :3000      │
                    └───────▲────────┘
                            │
                            │
                    ┌───────┴────────┐
                    │    Browser     │
                    └────────────────┘
```


서비스 요청은

`Browser → Next.js → Express → Flask`

순으로 전달되며, Express는 회원/세션 관리와 데이터 처리 및 외부 서비스 연동을 담당하고 Flask는 머신러닝 예측을 담당하도록 역할을 분리했습니다.

데이터 수집 및 적재는 GitHub Actions를 통해 자동화하고, 적재된 데이터는 MariaDB에 저장합니다. 이후 저장된 데이터를 기반으로 데이터 전처리 및 피처 구성을 수행하고 XGBoost 모델 학습에 활용합니다.


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

``` text
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

기상 조건뿐만 아니라 과거 산불 발생 이력과 최근 강수량, 건조 일수 등의
정보를 함께 활용하도록 구성했습니다.

### 학습 및 추론 구조

  단계            위치
  --------------- --------------------------------
  데이터 전처리   `etl/`
  모델 학습       `etl/ml/train_wildfire_xgb.py`
  모델 저장       `ml-service/models/`
  예측            `ml-service/predict/daily.py`
  Flask API       `ml-service/routes/`

학습된 XGBoost 모델은 `ml-service`에서 로드하여 예측에 사용합니다.

## 데이터 구성

### 산불 이력 데이터

MariaDB의 `forestfire_stats`를 주요 데이터 소스로 사용합니다.

``` text
MariaDB
forestfire_stats
       │
       ├── 산불 이력 조회
       ├── 모델 학습
       └── 지도 데이터 갱신
```

DB 사용이 어려운 경우 일부 데이터는 CSV를 활용할 수 있도록 fallback
구조를 구성했습니다.

### 기상 데이터

기상청 ASOS API를 활용하여 당일 기상 데이터를 가져오고, MariaDB에 저장된
시군구별 과거 기상 데이터를 활용하여 예측에 필요한 lag 및 통계 피처를
구성합니다.

주요 데이터는 다음과 같습니다.

-   평균 기온
-   강수량
-   평균 풍속
-   평균 상대습도
-   최근 강수량
-   건조 일수

### 데이터 흐름

``` text
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

-   산불 위험 지도
-   지역 및 산 검색
-   산불 예측 결과 표시
-   시나리오 예측
-   기상 정보 표시
-   로그인 UI
-   챗봇 UI
-   PDF 리포트 UI

Next.js Route Handler를 통해 Express Backend와 통신하도록 구성했습니다.

### Backend

`backend/`

Express와 TypeScript를 기반으로 API와 서비스 로직을 담당합니다.

주요 역할:

-   REST API
-   회원가입 및 로그인
-   세션 관리
-   Google / Kakao OAuth
-   Flask ML Service 연동
-   Gemini 챗봇 연동
-   산불 이력 데이터 갱신
-   예측 데이터 관리
-   PDF 리포트 요청 처리

### ML Service

`ml-service/`

Flask 기반의 머신러닝 서비스입니다.

주요 역할:

-   XGBoost 모델 로딩
-   일별 산불 예측
-   시나리오 예측
-   예측 결과 반환
-   PDF 리포트 생성 지원
-   예측 결과 DB 저장

머신러닝 영역을 웹 Backend와 별도의 서비스로 분리하여 모델과 웹
애플리케이션의 역할을 구분했습니다.

### ETL

`etl/`

웹서비스 요청과 분리된 오프라인 데이터 처리 영역입니다.

주요 역할:

-   산불 데이터 전처리
-   기상 데이터 전처리
-   데이터 분석
-   지도 데이터 생성
-   산 정보 수집
-   지역명 lookup 생성
-   머신러닝 모델 학습

주요 실행 파일:

``` text
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

## 폴더 구조

``` text
ForestFire/
│
├── frontend/
│   ├── public/data/
│   └── src/
│       ├── app/
│       ├── components/
│       ├── lib/
│       └── app/api/[...path]/
│
├── backend/
│   ├── data/
│   ├── migrations/
│   └── src/
│
├── ml-service/
│   ├── models/
│   ├── reference/
│   ├── predict/
│   ├── report/
│   └── routes/
│
├── etl/
│   ├── pipeline/
│   ├── analyze/
│   ├── map/
│   └── ml/
│
├── db/
├── db-archive/
├── .gitignore
└── README.md
```

`db/`와 `db-archive/`는 로컬에서 사용하는 대용량 데이터 및 ETL·분석
원본/중간 산출물을 관리하는 영역이며 Git에서는 제외합니다.

## 주요 API

### Express API

#### 지도 · 예측

``` text
GET  /api/health
GET  /api/map/data
GET  /api/map/admin/:level

POST /api/predict/daily
GET  /api/predict/scenario/baseline
POST /api/predict/scenario

POST /api/wildfires/sync
GET  /api/wildfires/sync/status
```

#### 회원

``` text
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

#### 챗봇 · 리포트

``` text
POST /api/chat
GET  /api/chat/history

GET  /api/report/daily
POST /api/report/pdf
GET  /api/report/download/:id
```

### Flask API

Flask API는 Express Backend에서 호출하는 내부 서비스로 사용합니다.

``` text
GET  /health

POST /predict/daily

GET  /predict/scenario/baseline
POST /predict/scenario

POST /report/pdf
```

## AI 챗봇

Gemini API를 활용하여 산불 예측 결과와 지역 정보를 기반으로 사용자의
질문에 답변하는 안내 챗봇을 구현했습니다.

챗봇은 Express Backend에서 처리하며, 당일 예측 API 결과를 우선 사용하고
필요한 경우 저장된 예측 데이터를 fallback으로 활용합니다.

회원 사용자의 경우 이전 대화 내역을 저장하고 다시 불러올 수 있도록
구성했습니다.

## PDF 리포트

로그인한 사용자는 산불 예측 결과를 기반으로 지역별 PDF 리포트를 생성할
수 있습니다.

``` text
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

생성된 PDF는 DB에 장기간 저장하지 않고 메모리에 임시 보관한 후 제한된
시간 동안 다운로드할 수 있도록 구성했습니다.

## 실행 방법

이 프로젝트는 Frontend, Backend, ML Service의 세 프로세스를 순서대로
실행하는 구조입니다.

### 1. Flask ML Service

``` powershell
cd ml-service

pip install -r requirements.txt

# PDF 리포트 기능 사용 시 최초 1회
playwright install chromium

python app.py
```

기본적으로 `5000` 포트를 사용합니다.

### 2. Express Backend

``` powershell
cd backend

npm install
npm run dev
```

기본적으로 `4000` 포트를 사용합니다.

### 3. Next.js Frontend

``` powershell
cd frontend

npm install
npm run dev
```

기본적으로 `3000` 포트를 사용합니다.

브라우저에서 다음 주소로 접속합니다.

``` text
http://localhost:3000
```

## 환경변수

실행에 필요한 API Key, DB 인증정보 및 OAuth Secret 등은 환경변수로
관리합니다.

주요 환경변수:

``` text
KMA_API_AUTH_KEY

DB_HOST
DB_PORT
DB_USER
DB_PASSWORD
DB_NAME

ML_HOST
ML_PORT
ML_SERVICE_URL

GEMINI_API_KEY

SESSION_SECRET

GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET

KAKAO_REST_API_KEY
KAKAO_CLIENT_SECRET

NEXT_PUBLIC_KAKAO_MAP_KEY
EXPRESS_URL
```

실제 인증정보 및 API Key는 Repository에 포함하지 않습니다.

## Docker

Frontend는 Next.js의 standalone output을 활용하여 Docker 환경에서도
실행할 수 있도록 구성했습니다.

``` powershell
cd frontend

docker compose --env-file .env.local up -d --build
```

Frontend 컨테이너는 3000 포트를 사용하며, Express와 Flask는 별도의
프로세스로 실행합니다.


## 주요 화면

- 좌측: FORESTFIRE ATLAS KOREA 브랜드, 지역·산 통합 검색, 위험 표시(당일 예측 / 사용자 지정 / 과거 이력), **최근 산불 발생** 피드(날짜만, 클릭 시 지도에 빨간 핀)
- 지도: SVG 행정구역 + 카카오 위성. 산 검색은 **파란 핀**, 산불 이력 클릭은 **빨간 핀**. 범례는 우측 하단(예측 모드: 산불위험지수 0~100 · 예측일 · AUC)
- 우측: 접을 수 있는 패널. 전체 산불 건수·갱신 날짜·갱신 버튼, 지역 미선택 시 전국 평균·최고 위험 시도 요약 → 선택 시 이력(날짜만)·산 상세(산림청 이미지). 로그인 시 지도 상단에 보고서 버튼
- 사용자 지정: 접속월부터 12개월(기본값 다음 달). 슬라이더 평년은 MariaDB `weather_daily_sigungu`에서 선택한 달의 전 기간·전국 시군구 일자료 평균. 건조·강풍 / 고온·건조 / 습함·비 많음은 같은 달 분포의 10·90분위(모드를 정의하는 변수만, 나머지는 평년). 예측은 `year`/`month`/`weather`를 그대로 전달
- 기상 카드: 선택 지역이 있으면 해당 시군구(또는 시도 평균) 기상

## 런타임 구조 (역할 분리)

| 프로세스 | 폴더 | 포트 | 역할 |
|----------|------|------|------|
| Next.js | `frontend/` | 3000 | UI · `/api` → Express 프록시 · 챗봇·로그인·보고서 모달 · 산 이미지 정적 파일 |
| Express | `backend/` | 4000 | 공개 API · 회원/세션 · 챗봇 · 산불이력 맵 갱신 · 보고서 게이트 · Flask 프록시 · DTO 화이트리스트 |
| Flask | `ml-service/` | 5000 | 예측 · PDF 렌더(Playwright) · localhost 전용 · 당일 예측 DB 스냅샷 |
| 배치 ETL | `etl/` | — | 전처리·학습·산 이미지·법정동 lookup (웹 요청에서 실행하지 않음) |

```
브라우저 → Next(:3000) ─Route Handler─→ Express(:4000) → Flask(:5000)
                                    ↘ map JSON · Gemini · 회원 DB
```

`/api/*` 는 `frontend/src/app/api/[...path]/route.ts` 가 Express로 넘깁니다.  
쿠키·`Set-Cookie`와 PDF의 `Content-Disposition`도 전달합니다.  
브라우저에는 Express가 필터한 DTO만 전달됩니다. 파이썬 stdout·모델 경로·API 키는 응답에 포함되지 않습니다.

**PDF 보고서:** 회원 세션 확인(Express) → `ml-service` Jinja2+Playwright로 **A4 가로** PDF 생성 → 메모리에 임시 보관(TTL 30분) 후 다운로드 URL 발급.  
표지(발행일·작성·닉네임 + 요약·게이지) 뒤 본문. 전국 리포트는 시군구 순위를 **상위 10·하위 5**만 넣고, 특정 지역 리포트는 해당 범위를 유지합니다.

## 데이터 소스

| 용도 | 우선 소스 | 비고 |
|------|-----------|------|
| 산불 이력 (맵 갱신·학습·분석) | MariaDB `forestfire_stats` | 실패 시 `refined_wildfire_data.csv` 폴백 |
| 예측용 당일 기상 | 기상청 ASOS API | `KMA_API_AUTH_KEY` |
| 예측용 lag 기상 (어제·그저께) | MariaDB `weather_daily_sigungu` | 실패 시 CSV 폴백 |
| 학습용 기상 | MariaDB 우선 | 동일 테이블 / CSV 폴백 |
| 지도 JSON | `frontend/public/data` + `backend/data` | 첫 로딩은 `/api/map/*`(`backend/data`). Express 불가 시에만 `frontend/public/data` 폴백. 웹 이력 갱신은 `backend/data`만 패치 |
| 챗봇·리포트용 당일 예측 스냅샷 | `backend/data/daily_ml_risk.json` | Express가 예측 API 성공 시 저장. Flask는 같은 결과를 MariaDB `daily_ml_risk_runs` / `daily_ml_risk_regions`에도 적재. 챗봇은 예측 API 우선, 실패 시 파일 폴백 |
| 산 썸네일 | `frontend/public/data/mountain-images/` | 오프라인 `etl/pipeline/fetch_mountain_images.py` (산림청 산정보 OpenAPI). 런타임에 API를 치지 않음 |
| 지역명 정규화 | `legal-dong-lookup.json` | UI는 `frontend/public/data`. Express는 `backend/data` → frontend 경로 폴백 |

회원/비회원은 **로그인 세션 유무**로만 구분합니다. 구독·결제 테이블은 사용하지 않습니다.  
로그인은 로컬(아이디/비밀번호) + 구글/카카오 OAuth를 지원하며, 유휴 30분 후 자동 로그아웃됩니다(연장 가능).

## 폴더 구조

```
ForestFire/
├── frontend/          Next.js UI (:3000, Docker standalone)
│   ├── public/data/   폴백 지도 JSON · legal-dong-lookup · mountain-images
│   └── src/           app · components · lib · app/api/[...path] 프록시
├── backend/           Express 공개 API (:4000, TypeScript)
│   ├── data/          지도·daily_ml_risk JSON 사본
│   ├── migrations/    챗봇 세션·소셜 로그인·당일 예측 테이블 SQL
│   └── src/
├── ml-service/        Flask (:5000, localhost)
│   ├── models/        XGBoost 모델 JSON
│   ├── reference/     시군구 hist · 관측소 매핑 CSV
│   ├── predict/       예측 엔진 + weather_db · fire_db · risk_snapshot_db
│   ├── report/        지역별 PDF (Jinja2 + Playwright)
│   └── routes/        health · predict · report
├── etl/               오프라인 ETL · 분석 · 학습 · 산 이미지
├── db/                로컬 대용량 CSV (Git 제외)
└── db-archive/        ETL·분석 원본·중간 산출물 (Git 제외)
```

## 웹 앱 실행 (3개 프로세스)

PowerShell — 터미널을 세 개 엽니다. **Flask → Express → Next** 순을 권장합니다.

### 1) Flask (`ml-service`)

```powershell
cd ml-service
# .env 에 KMA_API_AUTH_KEY · DB_* 입력 (아래 표 참고)
pip install -r requirements.txt
playwright install chromium   # PDF 보고서용 — 1회
python app.py
```

Linux 배포 시 Chromium 시스템 의존성·한글 폰트는 `ml-service/README.md` 참고.

### 2) Express (`backend`)

```powershell
cd backend
# .env 에 FRONTEND_ORIGIN · ML_SERVICE_URL · GEMINI_API_KEY · DB_* · SESSION_SECRET 등 입력
npm install
npm run dev
```

기동 시 당일 예측을 몇 번 재시도해 캐시에 올려 둡니다(Flask가 늦게 떠도 대비).

### 3) Next.js (`frontend`)

```powershell
cd frontend
# .env.local 에 NEXT_PUBLIC_KAKAO_MAP_KEY · EXPRESS_URL 입력
npm install
npm run dev
```

브라우저: http://localhost:3000  
헬스: http://localhost:4000/api/health

### Frontend Docker (선택)

`next.config.ts` 는 `output: "standalone"` 입니다. Lightsail 등에서 UI만 컨테이너로 올릴 때:

```powershell
cd frontend
docker compose --env-file .env.local up -d --build
```

빌드 인자로 `NEXT_PUBLIC_KAKAO_MAP_KEY` · `EXPRESS_URL` 이 들어갑니다. 호스트 80 → 컨테이너 3000. Express·Flask는 별도 프로세스입니다.

## 환경변수

| 위치 | 키 | 용도 |
|------|-----|------|
| `ml-service/.env` | `KMA_API_AUTH_KEY` | 기상청 ASOS (당일 예측) |
| `ml-service/.env` | `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | MariaDB (산불·lag 기상·당일 예측 스냅샷) |
| `ml-service/.env` | `ML_HOST`, `ML_PORT` | 바인딩 (기본 `127.0.0.1:5000`) |
| `ml-service/.env` | `FOREST_FIRE_SERVICE_KEY` | (선택) 레거시 OpenAPI 스크립트용 |
| `etl/.env` 또는 루트 `.env` | `FOREST_MOUNTAIN_SERVICE_KEY` | 산림청 산정보·산 이미지 수집 (오프라인 ETL만) |
| `backend/.env` | `PORT`, `FRONTEND_ORIGIN`, `ML_SERVICE_URL`, `PREDICT_CACHE_MS`, `DATA_DIR` | CORS · Flask URL · 예측 캐시 · 지도 데이터 폴더 |
| `backend/.env` | `GEMINI_API_KEY`, `GEMINI_MODEL`(선택) | 안내 챗봇 (`/api/chat`) |
| `backend/.env` | `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | 회원·챗봇·산불이력 동기화 (ml-service 와 동일 MariaDB 권장) |
| `backend/.env` | `SESSION_SECRET`, `SESSION_IDLE_MINUTES`(선택, 기본 30) | 로그인 세션 쿠키 서명 · 유휴 만료 |
| `backend/.env` | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | 구글 OAuth (미설정 시 버튼 비활성) |
| `backend/.env` | `KAKAO_REST_API_KEY`, `KAKAO_CLIENT_SECRET`(선택) | 카카오 OAuth |
| `backend/.env` | `OAUTH_REDIRECT_BASE`(선택) | OAuth 콜백 베이스. 기본값은 `FRONTEND_ORIGIN` |
| `frontend/.env.local` | `NEXT_PUBLIC_KAKAO_MAP_KEY`, `EXPRESS_URL` | 카카오 JS 키 · Express 주소 (`http://127.0.0.1:4000` 권장) |

카카오 개발자 콘솔에서 **JavaScript 키**를 쓰고, Web 플랫폼에 `http://localhost:3000` 을 등록해야 위성 지도가 표시됩니다.

구글/카카오 로그인을 쓰려면 각 콘솔에서 OAuth Redirect URI를 등록하세요:
- 구글: `http://localhost:3000/api/auth/google/callback`
- 카카오: **앱 → 플랫폼 키 → REST API 키** 하위에 `http://localhost:3000/api/auth/kakao/callback`

`KMA_API_AUTH_KEY` / `DB_*` / `FOREST_FIRE_SERVICE_KEY` / `FOREST_MOUNTAIN_SERVICE_KEY` / `GEMINI_API_KEY` / `SESSION_SECRET` / `GOOGLE_CLIENT_SECRET` / `KAKAO_*` 는 프론트 `.env.local`에 두지 마세요.

### backend와 frontend가 지도 데이터를 나눠 갖는 이유

`frontend`와 `backend`는 지도 JSON(`map-data.json`, `admin-*.json`)을 각자 폴더에 **따로** 둡니다.

- `backend/data` — 브라우저 첫 로딩(`/api/map/*`) · 웹 이력 갱신 · 챗봇 폴백
- `frontend/public/data` — Express가 꺼져 있을 때의 폴백 · ETL 원본 배포본 · 산 이미지 · 법정동 lookup

원본 지도 JSON은 `etl`이 만듭니다. 오프라인에서 `frontend/public/data`를 갱신할 때
`etl/paths.py`의 `sync_backend_data()`가 `backend/data`로도 복사합니다.

웹 **산불이력 갱신**(`POST /api/wildfires/sync`)은 Express가 MariaDB를 읽고
`backend/data`의 건수·색만 패치합니다. `frontend/public/data`는 건드리지 않습니다.
페이지를 새로고침해도 브라우저는 `/api/map/*`를 다시 읽으므로, 직전에 동기화한
건수·최근 이력이 유지됩니다.

당일 예측 스냅샷 `daily_ml_risk.json`은 Express가 `backend/data`에 저장합니다.

산 이미지 파일은 Next 정적 경로(`/data/mountain-images/{산코드}.jpg`)입니다. Git에는 `.gitkeep`만 두고 이미지는 제외합니다.

## 회원 · 챗봇 · 보고서 (요약)

| 기능 | 비회원 | 회원(로그인) |
|------|--------|----------------|
| 지도 · 당일/시나리오 예측 · 챗봇 Q&A | ✅ | ✅ |
| 챗봇 「이전 대화내역 불러오기」 | ❌ | ✅ |
| PDF 보고서 생성·다운로드 | ❌ | ✅ |
| 챗봇 대화 DB 저장 | DB 설정 시 게스트 세션 | `user_id`로 묶여 기기 무관 이어짐 |

### 회원가입 규칙

- **아이디:** 영문 소문자로 시작, 소문자+숫자만, 4~20자, 공백·특수문자 불가
- **비밀번호:** 8~20자, 영문 대문자/소문자/숫자/특수문자 중 2가지 이상 조합, 비밀번호 확인 일치 필수
- **소셜 로그인:** 구글/카카오 — **로그인 모달**에서는 기존 소셜 계정만 허용, **회원가입 모달**에서만 신규 생성 (비밀번호 없음, 닉네임 자동 배정). `intent=login|register`

### 유휴 세션

- 마지막 사용자 조작(클릭·키보드·스크롤) 기준 30분 (`SESSION_IDLE_MINUTES`)
- 자동 폴링(예측·동기화)은 연장하지 않음
- 만료 5분 전 안내 모달 (로그아웃 / 시간 연장)
- 만료되면 자동 로그아웃

### 챗봇 · PDF

- 챗봇: 예측 API(`runPredictDaily`) 우선 → 실패 시 `backend/data/daily_ml_risk.json`
- 로그인 회원은 최근 대화를 `user_id` 기준으로 Gemini 맥락에 포함 (게스트는 `sessionId`). 로그인 직후 게스트로 쓰던 `sessionId`는 계정 히스토리에 합쳐집니다
- 게스트 세션 ID는 브라우저 `localStorage`. HTTPS·localhost가 아니면 `crypto.randomUUID`가 없을 수 있어, 프론트에서 UUID v4 폴백으로 발급 (`frontend/src/components/ChatWidget.tsx`)
- UI는 열 때 인삿말만 보임. **「이전 대화내역 불러오기」는 로그인 회원만** 표시. 헤더에 닉네임·회원/게스트 뱃지
- 「보고서/PDF 만들어줘」: **로그인 필수**. `regionFocus`가 확신 있는 지명만 추출(잡음·예시·따옴표 문장 제외). 순서는 현재 문장 → 최근 유저 발화 → 어시스턴트. 「전국」도 가능. 없으면 지역을 되물은 뒤 Flask `POST /report/pdf`
- 보고서는 DB에 저장하지 않고, 생성 후 30분 TTL로 다운로드만 제공합니다
- 지도 **보고서** 모달의 PDF는 blob 다운로드(화면 유지). 모달은 상단 오버레이가 범례보다 위. 챗봇 PDF는 다운로드 링크
- 챗봇 창은 헤더 드래그로 위치 이동 가능 (플로팅 버튼은 우측 하단 고정)

자세한 API·폴더 구조는 `backend/README.md` · `frontend/README.md` · `ml-service/README.md` 참고.

## 예측 모델 (요약)

시군구×일 산불 발생 확률 — XGBoost.  
웹에서는 raw 확률(`ml_risk`) × 100을 **산불위험지수 (0~100)** 로 표시합니다.

**피처 (10):** `temp_avg`, `precip`, `wind_avg`, `humidity_avg`, `hist_fire_rate`, `hist_fire_count_365`, `dwi`, `precip_sum_7d`, `precip_sum_14d`, `dry_days`

| 단계 | 위치 |
|------|------|
| 학습 | `etl/ml/train_wildfire_xgb.py` (기상·산불: MariaDB 우선) |
| 추론 | `ml-service/predict/daily.py` |
| 산출물 | `ml-service/models/wildfire_xgb_*.json`, `ml-service/reference/sigungu_hist_state.csv` 등 |
| DB 스냅샷 | `daily_ml_risk_runs` + `daily_ml_risk_regions` (`backend/migrations/002_daily_ml_risk.sql`) |

CLI 예측:

```powershell
cd ml-service
python -m predict.daily --kma
```

당일 KMA 예측이 끝나면 Flask가 MariaDB에도 UPSERT 합니다(시나리오 예측은 넣지 않음). 테이블이 없거나 `DB_*`가 비어 있으면 로그만 남기고 예측 HTTP는 그대로 응답합니다.

## 산불 이력 갱신 (MariaDB)

웹: 우측 패널 헤더의 새로고침 버튼 클릭  
API: `POST /api/wildfires/sync` — Express가 MariaDB `forestfire_stats`를 읽어 `backend/data`의 건수·색을 패치합니다.
웹 첫 화면·F5도 같은 `/api/map/*`를 읽습니다. `frontend/public/data`는 Express 불가 시 폴백입니다.

오프라인에서 `frontend/public/data`까지 맞추려면:

```powershell
python etl/pipeline/sync_wildfire_history.py
```

(참고) 예전 공공데이터 OpenAPI 증분 스크립트는 `etl/pipeline/sync_wildfire_openapi.py` 에 남아 있으나, **웹 버튼·기본 동기화 경로는 DB**입니다.

## Express API (공개)

**맵 · 예측 · 동기화**

- `GET /api/health`
- `GET /api/map/data`
- `GET /api/map/admin/:level` (`sido` \| `sigungu` \| `emd`)
- `POST /api/predict/daily` — body: `{ source, force, date?, weather? }`
- `GET /api/predict/scenario/baseline?month=9` — 해당 월 평년·프리셋 기상 (`weather_daily_sigungu` 월평균 + 10·90분위)
- `POST /api/predict/scenario` — body: `{ year, month, weather: { temp_avg, humidity_avg, wind_avg, precip } }`  
  (UI는 접속월부터 12개월, 기본 다음 달. baseline API로 슬라이더를 채움)
- `POST /api/wildfires/sync` — MariaDB 산불 이력 → 맵 갱신
- `GET /api/wildfires/sync/status`

**회원** (로컬 아이디/비밀번호, 구글/카카오 OAuth, 유휴 30분 세션)

- `POST /api/auth/register` · `login` · `extend` · `logout`
- `GET /api/auth/me`
- `GET /api/auth/google` · `/kakao` · `/google/callback` · `/kakao/callback`  
  (`?intent=login|register`. 로그인은 기존 소셜 계정만, 회원가입에서만 신규 생성)

**챗봇 · 보고서**

- `POST /api/chat` — body: `{ message, sessionId? }` (비로그인 가능; 보고서 요청은 회원; 회원은 user 기준 히스토리; PDF 지역은 대화 맥락·`regionFocus`)
- `GET /api/chat/history` — 최근 대화 (회원=`user_id`, 게스트=`sessionId`). **UI 불러오기 버튼은 회원만**
- `GET /api/report/daily` — JSON 요약 (**회원**)
- `POST /api/report/pdf` — body: `{ regionQuery? }` → 다운로드 메타 (**회원**, 비우면 전국)
- `GET /api/report/download/:id` — PDF 바이너리 (**회원**, 메모리 TTL 30분, `Content-Disposition: attachment`)

## Flask API (내부)

- `GET /health`
- `POST /predict/daily` — Express만 호출
- `GET /predict/scenario/baseline?month=` — 월 평년·프리셋 기상. Express만 호출
- `POST /predict/scenario` — Express만 호출
- `POST /report/pdf` — body: `{ region }` — Express만 호출 (PDF 바이트)

## 배치 파이프라인 (오프라인)

자세한 설명은 `etl/README.md` 참고.

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
cd ml-service; python -m predict.daily --kma
```

경로 상수는 `etl/paths.py` 한곳에서 관리합니다.  
산불 원본 로드는 `etl/pipeline/load_wildfire_history.py` (DB 우선)를 공통으로 씁니다.

## 프로젝트를 통해 배운 점

이번 프로젝트에서는 머신러닝 모델을 만드는 것뿐만 아니라 모델의 결과가
실제 웹서비스의 기능으로 연결되는 전체 흐름을 경험했습니다.

특히 데이터와 모델을 구현한 뒤에는

``` text
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

또한 프로젝트 진행 과정에서 AI 개발 도구를 활용하면서 생성된 코드의
동작을 그대로 사용하는 것보다 API 호출 구조와 데이터 흐름을 직접
확인하고 코드의 역할을 이해하는 것이 중요하다는 점을 경험했습니다.
