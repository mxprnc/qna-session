---
name: qna-session
description: "Manages a recursive Q&A requirements gathering session. Prompts the user for Q1, Q2, Q3... Q[n] sequentially, stores Q&A text inside a dynamically generated qna{N}.md session log, compiles HTML visualizations, and supports exporting logs to custom paths in various formats (HTML/Markdown)."
---

# Q&A Session Skill Guide

This skill implements a sequential Q&A process for requirements gathering, provides export commands to convert/copy results, and explains its own usage on demand.

## Workflow

### 1. Initialization
When the user requests to start a Q&A session:
```bash
python3 .agents/skills/qna-session/scripts/session_manager.py init
```
*   Save the returned `{num}` from `SESSION_INITIALIZED:{num}`.
*   Prompt: **"Q&A 세션 {num}이 시작되었습니다. 첫 번째 질문(Q1)을 남겨주세요. 종료하려면 '종료' 또는 '끝'이라고 말씀해 주세요."**

### 2. Q&A Loop
1.  Provide a detailed answer to the user's question.
2.  Append it to the session:
    ```bash
    python3 .agents/skills/qna-session/scripts/session_manager.py append --num {num} --q "{question_text}" --a "{answer_text}"
    ```
3.  Prompt: **"다음 질문을 입력해 주세요. (세션을 종료하려면 '종료' 또는 '끝'을 입력하세요.)"**

### 3. Finalization
When the user says "종료" or "끝":
1.  Finalize the active session to generate default reports:
    ```bash
    python3 .agents/skills/qna-session/scripts/session_manager.py finalize --num {num}
    ```
2.  Inform the user of files created in the `working/` folder.
3.  **Prompt the user for Export:** "Q&A 세션이 종료되었으며 질의응답 기록 문서가 안전하게 보관되었습니다. 이 문서를 가독성이 높은 HTML 대시보드로 변환해 드릴까요? (요약 목차 위주의 'overall' 타입과 상세 질의응답 기록 위주의 'raw' 타입 중 원하시는 형태를 선택해 주세요.)"

---

## 🚀 Exporting and Compiling Results

This skill handles converting raw Q&A logs into styled HTML/Markdown formats. The agent must parse CLI-style commands or conversational instructions into script invocations:

```bash
python3 .agents/skills/qna-session/scripts/session_manager.py export --format {fmt} --type {type} [--num {num} | --session_file {source_file}] [--output {out_file}]
```

### Parameters

*   `--format`: `html` (styled webpage), `md` (restructured report), or `raw` (exact raw backup).
*   `--type`: `overall` (TOC/summary topic layout) or `raw` (sequential detailed layout).
*   `--session_file`: Path to any raw session markdown file to parse.
*   `--output`: Target destination path.
    *   **Rule 1:** If no output directory/path is specified, default to the **Project Root** directory.
    *   **Rule 2:** If the target filename already exists, the script automatically renames it (e.g. `{filename}-1.{ext}`, `{filename}-2.{ext}`).
    *   **Rule 3:** The script outputs the actual physical output path upon success in `EXPORT_SUCCESS:{physical_path}` format. Read this line to locate the saved file.

---

## 📖 Sub-Skill: Self-Explanation & Guide (도움말 기능)

When the user asks for help, guides, explanations, or details on how this skill works (e.g., *"qna-session 사용법 알려줘"*, *"스킬 설명해줘"*, *"qna-session:help"*):

1.  **Read Local Documentation:** Read the [README.md](./README.md) inside the skill folder.
2.  **Generate a Presentation Guide:** Present a concise and clear summary of the skill's features directly in the chat or as a custom markdown report, introducing:
    *   How to start a Q&A session and the sequential flow.
    *   The advanced HTML visual system (PrismJS Syntax highlighting, Floating Copy buttons, Right-aligned sidebar navigation TOC, Client-side Search).
    *   Dynamic exporting mechanisms (HTML dashboards, structured Markdown reports, raw text streams) along with the default project root destination and duplicate filename prevention algorithms.
3.  Avoid copy-pasting the raw README code; adapt it to the user's query context in a friendly, professional manner.
