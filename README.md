# Q&A Session CLI Skill & Manager

이 리포지토리는 사용자의 요구사항 정의 및 Q&A 세션 진행 과정을 순차적으로 기록하고, 세션 종료 시 자동으로 프리미엄 스타일의 HTML 결과물(상세 내역 및 종합 대시보드 리포트)로 변환해 주는 **qna-session 커스텀 스킬**을 프로젝트에 탑재하고 있습니다.

---

## 📂 프로젝트 구조

```
├── .agents/
│   └── skills/
│       └── qna-session/
│           ├── SKILL.md            # 에이전트의 Q&A 세션 가이드 및 제어 지침서
│           ├── README.md           # 스킬 가이드 문서
│           └── scripts/
│               └── session_manager.py  # 세션 관리 및 컴파일/익스포트 엔진
└── README.md                       # 프로젝트 메인 가이드 (본 문서)
```

---

## 🛠️ qna-session 스킬 소개

`qna-session`은 사용자와 에이전트 간의 인터랙티브한 질의응답을 순차적으로 기록하여 요구사항을 정교화하고, 이를 다양한 형태(HTML, Markdown, Raw 백업)로 자동 변환 및 익스포트(Export)하는 에이전트 CLI용 커스텀 스킬입니다.

### 🌟 주요 기능
1. **순차적 Q&A 기록 및 관리**:
   - `session_manager.py` 스크립트를 통해 세션 초기화(`init`), 내용 추가(`append`), 세션 종료 및 컴파일(`finalize`)의 라이프사이클을 통제합니다.
2. **프리미엄 컴파일러 엔진**:
   - 세션이 종료되면 `working/` 디렉터리에 자동으로 두 종류의 다크 테마 HTML 결과물이 컴파일됩니다:
     - `qna{번호}.html`: 검색, 복사 및 PrismJS 코드 하이라이팅이 적용된 상세 카드 레이아웃.
     - `qna{번호}-overall.html`: 사이드바 테이블 오브 콘텐츠(TOC)와 클라이언트 측 검색 기능이 탑재된 구조화된 종합 보고서.
3. **유연한 데이터 익스포트**:
   - 저장된 세션 로그(`qna{번호}.md` 등)나 기타 외부 마크다운 리포트를 원하는 경로와 이름으로 익스포트할 수 있습니다.
   - **중복 방지 알고리즘**: 지정한 파일명이 이미 존재할 경우 덮어쓰지 않고 `-1`, `-2` 접미사를 붙여 안전하게 순차 저장합니다.

---

## 🚀 사용 가이드

### 1. 에이전트를 통한 대화형 실행
에이전트와의 채팅창에 자연스러운 한국어 문장으로 태스크를 요청할 수 있습니다:
- **세션 시작**: `"Q&A 세션 시작해줘"` 또는 `"qna-session 시작"`
- **세션 종료**: 세션 진행 중 `"종료"` 또는 `"끝"` 입력
- **익스포트 요청 예시**:
  - *"qna-session 스킬로 working/qna1.md 파일을 종합한 overall html로 프로젝트 디렉터리의 result.html에 정리해줘."*
  - *"세션 3 마크다운 내역을 요약 마크다운 보고서로 포맷팅해서 프로젝트 폴더의 report.md로 내보내줘"*

### 2. 세션 매니저 CLI 직접 실행

세션 관리 스크립트([session_manager.py](file://.agents/skills/qna-session/scripts/session_manager.py))는 터미널 CLI로도 직접 가동할 수 있습니다.

#### ① 세션 시작 (Init)
```bash
python3 .agents/skills/qna-session/scripts/session_manager.py init
```
*성공 시 `SESSION_INITIALIZED:{세션번호}`가 반환됩니다.*

#### ② Q&A 내용 추가 (Append)
```bash
python3 .agents/skills/qna-session/scripts/session_manager.py append --num {세션번호} --q "{질문내용}" --a "{답변내용}"
```

#### ③ 세션 종료 및 컴파일 (Finalize)
```bash
python3 .agents/skills/qna-session/scripts/session_manager.py finalize --num {세션번호}
```
*종료 시 `working/` 폴더 하위에 마크다운 원본과 HTML 뷰 파일들이 컴파일됩니다.*

#### ④ 세션 익스포트 및 변환 (Export)
```bash
python3 .agents/skills/qna-session/scripts/session_manager.py export --format {html|md|raw} --type {overall|raw} [--num {세션번호} | --session_file {대상마크다운파일}] [--output {출력파일명}]
```
- `--format`: `html` (CSS 스타일 적용 웹페이지), `md` (마크다운 보고서), `raw` (원본 데이터 백업)
- `--type`: `overall` (사이드바/TOC 레이아웃), `raw` (순차 상세 레이아웃)
- `--output`을 생략하거나 디렉터리 경로 없이 파일명만 적으면, 기본적으로 **프로젝트 루트 디렉터리**에 저장됩니다.
- 이미 같은 이름의 파일이 존재할 경우 파일명 중복을 감지하고 숫자를 덧붙여 안전하게 다중 백업합니다.

---

## 🎨 산출물 예시 및 컴파일 사양

- **상세 내역 카드 뷰 (`qna{번호}.html`)**:
  - 현대적이고 반응성이 뛰어난 다크 모드 카드 레이아웃
  - PrismJS를 사용한 소스 코드 구문 강조(Syntax Highlighting)
  - 개별 문단/코드 블록용 플로팅 복사 단추
  - 대용량 텍스트 대응 클라이언트 사이드 검색(Search) 및 하이라이트 기능
- **종합 요약 뷰 (`qna{번호}-overall.html`)**:
  - 화면 오른쪽에 고정된 동적 네비게이션 목차(TOC Sidebar)
  - 리드미 및 핵심 주제별 접고 펼치기(Accordion/Details) 또는 구조화된 섹션 레이아웃

---

## 🚨 에러 및 예외 해결 안내

- **Q&A 원본 파일을 찾을 수 없는 경우 (`ERROR: Source session markdown not found...`)**:
  - 지정한 세션 번호에 대응하는 `.agents/skills/qna-session/working/qna{번호}.md` 파일이 유실되었거나 경로가 지정되지 않았는지 확인해 주세요.
  - 외부 파일을 익스포트할 때도 해당 파일의 절대/상대 경로가 맞는지 체크해 주셔야 합니다.
