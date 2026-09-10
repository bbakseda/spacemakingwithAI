# [계획서] Gemini API 기반 학교 공간재구조화 설계 웹 애플리케이션 구축

기존에 작업하신 다른 폴더들은 전혀 건드리지 않고, 오직 이번 작업을 위해 **신규 전용 폴더**를 독립적으로 생성하여 완전히 격리된 환경에서 프로젝트를 구축합니다.

---

## 1. 신규 작업 폴더 안내

> [!IMPORTANT]
> **신규 작업 경로:** `g:\내 드라이브\코딩 작업물\학교_공간재구조화_AI설계`
> - 기존 작업 폴더(예: `건물 설계 앱`, `grade_processor` 등)는 전혀 수정하거나 변경하지 않습니다.
> - 모든 코드와 설정 파일(`backend/`, `frontend/`, `.env`, `README.md`)은 위 신규 폴더 내부에만 생성됩니다.

---

## 2. 프로젝트 아키텍처 및 세부 설계

```
g:\내 드라이브\코딩 작업물\학교_공간재구조화_AI설계\
├── backend/
│   ├── main.py                # FastAPI 서버, CORS, /api/generate-layout 엔드포인트
│   ├── gemini_service.py      # google-genai 최신 SDK, Structured Output 스키마 정의
│   ├── requirements.txt       # fastapi, uvicorn, google-genai, pydantic, python-dotenv 등
│   └── .env.example           # GEMINI_API_KEY 예시 설정 파일
├── frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json           # React, Vite, Lucide-react, Tailwind CSS, Canvas 라이브러리
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   └── src/
│       ├── App.jsx            # 3열 반응형 레이아웃 통합 컴포넌트
│       ├── main.jsx
│       ├── index.css          # Tailwind CSS 지시문
│       └── components/
│           ├── InputPanel.jsx    # 좌측 패널: 공간유형, 가로/세로, 요구사항 텍스트 입력 및 생성 버튼
│           ├── CanvasViewer.jsx  # 중앙 패널: 2D 그리드(1m 격자) + 구역(Zone) 블록 드래그&드롭 캔버스
│           └── ReportPanel.jsx   # 우측 패널: 교육적 의도, 안전/동선 분석 렌더링 & JSON 다운로드
├── .env                       # 최상위 GEMINI_API_KEY 설정 파일
└── README.md                  # 설치 및 원클릭 실행 가이드 (Windows 구동 가이드 포함)
```

---

## 3. 핵심 기능 구현 세부 계획

### 1) 백엔드 (FastAPI + google-genai)
- **SDK & 모델:** 최신 `google-genai` 라이브러리 활용, 모델 `gemini-2.5-flash` 지정
- **엔드포인트:** `POST /api/generate-layout`
  - 요청: `room_type`(공간유형), `width_m`(가로m), `height_m`(세로m), `user_requirements`(정성적 요구사항)
  - 응답 스키마 (Pydantic Structured Output):
    - `zones`: `[{ id, name, purpose, color_hex, x, y, width, height, required_furniture: [] }]`
      - 가로/세로 한도 내 배치 및 유의미한 구역 분할 보장
    - `educational_rationale`: 2022 개정 교육과정 및 공간 재구조화 철학 반영 설계 근거 (Markdown)
    - `safety_and_circulation`: 동선 안전성, 대피로, 유니버설 디자인 고려사항
- **예외 처리 & Mock 모드 지원:** 유효한 GEMINI_API_KEY가 없을 경우에도 UI와 캔버스를 즉시 체험해볼 수 있는 안전 폴백(Fallback Mock Layout) 지원

### 2) 프론트엔드 (React + Vite + 2D Interactive Canvas & Standalone HTML)
- **3열 반응형 대시보드 레이아웃:**
  - **좌측 (설정 & 요구사항 입력):**
    - 공간 유형 프리셋: 지능형 과학실, 홈베이스, 메이커스페이스, 융합미디어실, 가변형 교실 등
    - 교실 치수 입력 (가로 m, 세로 m)
    - 풍부한 기본 예시 시나리오가 담긴 텍스트 에어리어
    - AI 생성 버튼 (스피너 로딩 인디케이터)
  - **중앙 (인터랙티브 2D 공간 캔버스):**
    - 1m 단위 격자선(Grid) 및 미터 눈금자
    - 교실 전체 크기에 맞는 뷰포트 자동 스케일링(Scale fit)
    - 각 공간 블록(Zone) 직사각형 색상 렌더링, 구역명, 크기(m x m), 면적($m^2$) 표시
    - **실시간 마우스 드래그 앤 드롭 이동:** 사용자가 원하는 위치로 직접 블록을 옮겨 커스텀 배치 가능
    - 블록 클릭 시 상세 가구 목록 및 목적 툴팁/하이라이트
  - **우측 (교육적 설계 근거 & 리포트):**
    - 교육적 효과 및 공간 철학 리포트
    - 안전 및 동선 분석 결과 카드
    - **[배치 데이터 JSON 다운로드]** 및 **[인터랙티브 HTML 파일로 내보내기/저장]** 기능
- **✨ 최종 단독 실행형 인터랙티브 HTML (`standalone_app.html`):**
  - 복잡한 서버 설치 없이도 **브라우저에서 더블 클릭만으로 즉시 열어서 만질 수 있는 완성형 단독 HTML 파일**을 함께 제공합니다.
  - 브라우저 상에서 직접 Gemini API 호출(API 키 입력)하거나 백엔드와 연결할 수 있으며, 2D 캔버스 드래그 조작, 수정, 결과 HTML 저장까지 브라우저 하나로 완벽하게 제어할 수 있습니다.

---

## 4. 검증 계획

### 로컬 실행 검증
1. 신규 폴더 `학교_공간재구조화_AI설계` 내 파일 일체 생성 확인
2. 백엔드 스키마 유효성 및 API 모듈 무결성 검증
3. 프론트엔드 컴포넌트 렌더링 및 2D 캔버스 드래그 인터랙션 로직 검증
4. 백엔드/프론트엔드 각각의 원클릭 실행 스크립트(`run_backend.bat`, `run_frontend.bat`) 및 통합 가이드 제공

---

## 사용자 확인 요청

- 작업 폴더명으로 **`학교_공간재구조화_AI설계`**가 적절한지, 또는 선호하시는 특정 폴더명이 있으시면 말씀해주세요.
- 확인해주시면 기존 폴더는 일체 건드리지 않고 이 새 폴더 안에 모든 풀스택 코드를 완성도 높게 구현하겠습니다.
