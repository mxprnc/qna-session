# qna-session Skill (Q&A 세션 및 익스포트 매니저)

사용자의 요구사항 정의 및 Q&A 세션 진행 과정을 순차적으로 기록하고, 세션 종료 시 자동으로 프리미엄 스타일의 HTML 결과물(상세 내역 및 종합 대시보드 리포트)로 변환해 주는 에이전트 CLI용 커스텀 스킬입니다. 본 스킬은 세션 데이터의 포맷 변환 및 익스포트(Export) 기능이 통합되어 있습니다.

---

## 📂 폴더 구조 및 구성 요소

*   `SKILL.md`: 에이전트가 해당 스킬을 인식하고 Q&A 세션 진행 및 파일 익스포트 제어 흐름을 유도하는 지침서입니다.
*   `README.md`: 본 스킬의 사용 방법을 기록한 가이드 문서입니다.
*   `scripts/session_manager.py`: 세션 개시, 내용 추가, HTML 컴파일 및 파일 익스포트를 모두 담당하는 핵심 파이썬 스크립트입니다.
*   `working/`: 세션 진행 중 기본적으로 생성되는 마크다운 및 HTML 기본 컴파일 결과물들이 저장되는 폴더입니다.
    *   `qna{번호}.md`: 원본 Q&A 내역 마크다운 파일
    *   `qna{번호}.html`: 다크 테마 카드 레이아웃의 상세 Q&A 내역 웹페이지 (검색/복사 지원)
    *   `qna{번호}-overall.html`: 사이드바 네비게이션이 탑재된 종합 분석 보고서 웹페이지 (검색/복사 지원)

---

## 🚀 사용 방법

### 1. Q&A 세션 가동 및 기록

에이전트에게 **"Q&A 세션 시작해줘"** 혹은 **"qna-session 시작"**이라고 입력하면 세션이 생성됩니다.
질의응답 중 **"종료"** 또는 **"끝"**을 선언하면 Q&A가 마무리되고 `working/` 아래에 기본 HTML 웹뷰가 자동 컴파일됩니다.

---

### 2. 세션 데이터 변환 및 익스포트 (Export)

기존 세션 데이터(또는 외부 마크다운 파일)를 특정한 목적지에 원하는 포맷으로 변환해 내보낼 때 사용합니다.

#### CLI 실행 구문
```bash
python3 scripts/session_manager.py export --format {html|md|raw} --type {overall|raw} [--num {세션번호} | --session_file {대상파일경로}] [--output {출력경로}]
```

#### 💡 파일 익스포트 규칙
1.  **기본 저장 경로:** `--output`에 디렉터리 경로를 생략하거나 파일명만 적으면, 기본적으로 **프로젝트 루트 디렉터리**(`.../project-html-explorer/`)에 파일이 저장됩니다.
2.  **파일명 중복 방지:** 내보내려는 대상 파일명이 이미 존재할 경우, 단순 덮어쓰기하는 대신 `{중복파일명}-1.{확장자}` (또는 계속 존재 시 `-2`, `-3`으로 순차 숫자 증가) 형태로 충돌을 방지하며 안전하게 저장합니다.
3.  **실제 저장 경로 피드백:** 스크립트 실행이 성공하면 콘솔에 `EXPORT_SUCCESS:{실제물리경로}`를 출력합니다.

---

## 🎨 사용 예시

### ① 자연어 기반 에이전트 요청 예시 (대화형 실행 권장)
에이전트와의 채팅창에 자연스러운 문장으로 태스크를 요청할 수 있습니다:

*   **HTML 대시보드 변환 및 루트 저장 예시:**
    > *"qna-session 스킬로 working/qna1.md 파일을 종합한 overall html 로 변환하려고 해. 해당 파일은 현재 프로젝트 디렉터리 내에 result.html 에 정리해줘."*
    *   (결과: 프로젝트 루트 디렉터리에 `result.html` 또는 중복될 경우 `result-1.html`로 컴파일되어 파일 클릭 링크가 반환됩니다.)
*   **특정 세션 요약 보고서 마크다운 생성 예시:**
    > *"세션 3 마크다운 내역을 요약 마크다운 보고서로 포맷팅해서 프로젝트 폴더의 report.md로 내보내줘"*
*   **외부 파일 HTML 컴파일 예시:**
    > *"_worklog/requirements.md 파일을 읽어와서 우측 사이드바가 있는 상세 HTML로 변환해서 루트에 정리해줘"*
*   **백업 복사본 추출 예시:**
    > *"세션 2의 원본 마크다운 파일을 docs/backup_session2.md에 백업 복사해두고 싶어"*

---

### ② CLI 직접 실행 예시
*   **세션 2의 전체 내역을 종합 보고서 HTML로 프로젝트 루트의 `result.html`에 저장:**
    ```bash
    python3 scripts/session_manager.py export --num 2 --format html --type overall --output result.html
    ```
*   **외부 폴더의 마크다운 파일(`doc/qna3.md`)을 읽어와서 종합 보고서 마크다운 문서(`.md`)로 프로젝트 루트에 저장:**
    ```bash
    python3 scripts/session_manager.py export --session_file doc/qna3.md --format md --type overall
    ```
*   **세션 1 원본 마크다운 파일을 특정 백업 폴더로 복사:**
    ```bash
    python3 scripts/session_manager.py export --num 1 --format md --type raw --output backup/qna1_raw.md
    ```

---

## 🚨 에러 및 예외 처리

### 1. 대상 파일이 존재하지 않는 경우
지정한 세션 번호(`--num`) 또는 파일 경로(`--session_file`)가 유효하지 않거나 유실된 경우 아래와 같이 처리됩니다.

*   **콘솔 오류 코드:**
    ```bash
    ERROR: Source session markdown not found: /Users/.../.agents/skills/qna-session/working/qna99.md
    ```
*   **에이전트 대응 메시지:**
    에이전트는 내부적으로 이 에러 코드를 감지하고 사용자에게 다음과 같이 친절하게 안내합니다:
    > ❌ **"지정하신 Q&A 세션 소스 파일(또는 번호)을 찾을 수 없습니다. 경로가 올바른지 다시 확인해 주시기 바랍니다."**
