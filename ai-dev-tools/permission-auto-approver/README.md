# Claude Code 무인 실행 설정 (권한 전부 허용 + 질문은 권장안)

> 한 줄 요약: Claude Code가 권한을 물으면 모두 허용하고, 선택 질문을 하면 권장안을 고른 것으로 처리해, 사람이 화면 앞에 없어도 작업이 멈추지 않게 하는 설정 묶음(설정 파일 + 훅 스크립트 3개 + 지시문)을 만든다.

- 카테고리: ai-dev-tools
- 상태: 구체화 중
- 작성일: 2026-11-12
- 구현 리포지터리: -

## 배경 / 문제

- Claude Code로 긴 작업을 맡기면 `npm test`, `git status` 같은 명령에도 권한 확인 창이 떠서 작업이 멈춘다.
- 선택지가 있으면 Claude가 질문 창(AskUserQuestion)을 띄우거나, 응답 끝에 "진행할까요?"라고 묻고 멈춘다.
- 사람이 계속 붙어 있어야 해서 자리를 비우거나 밤새 작업을 돌릴 수 없다.

## 결정 사항

- **모든 권한 요청을 허용한다.** 요청별 판정(규칙·AI 판정)은 하지 않는다.
- **질문은 권장안을 수용한다.** 권장안이 표시되지 않은 질문은 첫 번째 선택지를 고른다.
- **외부 메신저(Slack 등)와 원격 승인은 쓰지 않는다.**
- 그 대신 위험은 **실행 환경을 격리**하는 것으로 줄이고, 자동으로 정한 내용은 **결정 로그**로 남겨 나중에 사람이 검토한다.

## 목표

- 권한 확인 창 때문에 작업이 멈추지 않는다.
- 질문 창과 "할까요?" 식의 텍스트 질문 때문에 작업이 멈추지 않는다.
- 자동으로 고른 답과 허용한 요청이 파일로 남아 작업 후 검토할 수 있다.

## 범위 밖 (Non-goals)

- 요청별 위험도 판정, 허용·거부 규칙 관리.
- Slack·메신저·모바일 원격 승인.
- 화면 클릭 자동화(컴퓨터 사용 모델, xdotool 등). 아래 방식으로 모두 처리되므로 쓸 이유가 없다.

## 구현 방법

멈추는 지점이 네 군데이고, 각각 다른 장치로 막는다.

| 멈추는 지점 | 막는 장치 | 설명 |
|---|---|---|
| 1. 도구 권한 확인 | 권한 모드 `bypassPermissions` | 권한 확인 없이 모든 도구를 실행한다. CLI 옵션 `--dangerously-skip-permissions`와 같다 |
| 2. 그래도 남는 권한 확인 | `PermissionRequest` 훅 → 항상 allow | 모드로 빠지지 않는 확인 창(계획 승인 등)을 잡는 안전망 |
| 3. 질문 창(AskUserQuestion) | `PreToolUse` 훅(matcher `AskUserQuestion`) | 권장안을 골라 Claude에게 돌려준다 |
| 4. 텍스트로 묻고 끝내기("진행할까요?") | `Stop` 훅 + CLAUDE.md 지시문 | 질문으로 끝나면 "권장안으로 계속하라"며 종료를 한 번 막는다 |

### 1. 설정 파일 (`.claude/settings.json`)

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  },
  "hooks": {
    "PermissionRequest": [
      { "matcher": "*", "hooks": [{ "type": "command", "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/allow_all.py\"" }] }
    ],
    "PreToolUse": [
      { "matcher": "AskUserQuestion", "hooks": [{ "type": "command", "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/answer_recommended.py\"" }] }
    ],
    "Stop": [
      { "hooks": [{ "type": "command", "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/no_question_stop.py\"" }] }
    ]
  }
}
```

- 무인 실행(`claude -p "작업"`)이면 `--permission-mode bypassPermissions` 또는 `--dangerously-skip-permissions`를 붙인다.
- root 계정에서는 이 모드가 거부될 수 있다. 컨테이너 안이면 일반 계정으로 실행한다.

### 2. `allow_all.py` — 남는 권한 확인을 모두 허용

```python
import json, sys, datetime
req = json.load(sys.stdin)
with open(".claude/decision-log.jsonl", "a", encoding="utf-8") as f:
    f.write(json.dumps({"t": datetime.datetime.now().isoformat(), "type": "permission",
                        "tool": req.get("tool_name"), "input": req.get("tool_input")}, ensure_ascii=False) + "\n")
print(json.dumps({"hookSpecificOutput": {"hookEventName": "PermissionRequest",
                                          "decision": {"behavior": "allow"}}}))
```

### 3. `answer_recommended.py` — 질문 창에 권장안으로 답하기

질문 창의 선택지 중 이름에 "(Recommended)"나 "권장"이 붙은 것을 고르고, 없으면 첫 번째 선택지를 고른다. Claude는 권장안을 첫 번째 선택지로 두고 "(Recommended)"를 붙이는 관례가 있다.

```python
import json, sys, datetime
req = json.load(sys.stdin)
questions = req.get("tool_input", {}).get("questions", [])

def pick(q):
    opts = q.get("options", [])
    for o in opts:
        if "Recommended" in o["label"] or "권장" in o["label"]:
            return o["label"]
    return opts[0]["label"] if opts else "권장안으로 진행"

answers = {q["question"]: pick(q) for q in questions}
with open(".claude/decision-log.jsonl", "a", encoding="utf-8") as f:
    f.write(json.dumps({"t": datetime.datetime.now().isoformat(), "type": "question",
                        "answers": answers}, ensure_ascii=False) + "\n")

reason = ("사용자는 자리에 없어 답할 수 없다. 아래 답을 사용자가 고른 것으로 보고 질문 없이 계속 진행하라.\n"
          + "\n".join(f"- {q} → {a}" for q, a in answers.items()))
print(json.dumps({"hookSpecificOutput": {"hookEventName": "PreToolUse",
                                          "permissionDecision": "deny",
                                          "permissionDecisionReason": reason}}))
```

- 질문 창을 띄우지 않고(deny) 거부 이유에 고른 답을 담아 돌려준다. Claude는 거부 이유를 읽고 그 답으로 작업을 이어 간다. 버전에 덜 의존하는 방식이다.
- 다른 방식으로, `permissionDecision: "allow"`와 함께 `updatedInput`에 `answers`(질문 → 고른 선택지)를 채워 질문 창이 답을 받은 것처럼 처리할 수도 있다. 사용 중인 Claude Code 버전에서 지원하는지 먼저 확인해야 한다.

### 4. `no_question_stop.py` — "진행할까요?"로 끝내지 못하게

```python
import json, sys, re
req = json.load(sys.stdin)
if req.get("stop_hook_active"):        # 이미 한 번 계속시켰으면 그대로 끝낸다 (무한 반복 방지)
    sys.exit(0)
last = ""
with open(req["transcript_path"], encoding="utf-8") as f:
    for line in f:
        e = json.loads(line)
        if e.get("type") == "assistant":
            for c in e.get("message", {}).get("content", []):
                if c.get("type") == "text":
                    last = c["text"]
tail = last.strip()[-200:]
if re.search(r"(\?|할까요|하시겠습니까|원하시나요|알려 ?주세요)\s*$", tail):
    print(json.dumps({"decision": "block",
                      "reason": "사용자는 답하지 않는다. 방금 질문에 대해 권장안을 골라 결정 로그에 남기고 계속 진행하라. 더 할 일이 없으면 결과만 보고하고 끝내라."}))
```

### 5. CLAUDE.md 지시문 (질문 자체를 줄이기)

```markdown
## 무인 실행 규칙
- 사용자는 작업 중 응답하지 않는다. 질문하지 말고, 선택이 필요하면 권장안을 골라 진행한다.
- 고른 내용은 `.claude/decision-log.md`에 "결정 / 선택지 / 고른 이유" 한 줄로 남긴다.
- 끝낼 때 자동으로 정한 결정 목록을 보고에 포함한다.
```

### 6. 실행 환경 (권장)

- 전부 허용이므로 **격리된 환경에서만 쓴다**: 개발 컨테이너·VM·클라우드 세션. 개인 PC의 홈 폴더 전체나 운영 서버 접속 정보가 있는 환경에서는 쓰지 않는다.
- 운영 계정, 클라우드 관리자 키, 사내 중요 시스템 접속 정보를 그 환경에 두지 않는다.
- 작업 브랜치에서만 작업하고, 작업 전 커밋해 두어 언제든 되돌릴 수 있게 한다.

## 사용 시나리오

1. 개발자가 격리된 개발 컨테이너에서 위 설정이 들어간 저장소를 열고 Claude Code에 긴 작업을 맡긴 뒤 자리를 비운다.
2. 권한 확인은 모두 자동 허용되고, Claude가 선택지를 물으면 권장안이 자동으로 선택된다.
3. Claude가 "이렇게 진행할까요?"로 끝내려 하면 Stop 훅이 한 번 계속시킨다.
4. 돌아온 개발자는 결과와 `.claude/decision-log.jsonl`·`decision-log.md`의 자동 결정 목록을 검토하고, 마음에 들지 않는 결정만 다시 지시한다.

## 핵심 기능

- [ ] 권한 모드 bypassPermissions 설정
- [ ] PermissionRequest 전부 허용 훅
- [ ] AskUserQuestion 권장안 자동 응답 훅
- [ ] 질문으로 끝나는 응답을 계속시키는 Stop 훅
- [ ] 무인 실행 CLAUDE.md 지시문
- [ ] 결정 로그(허용한 요청, 고른 답)
- [ ] 설치 스크립트(위 파일을 저장소에 복사)

## 기술 구상

- 기술 스택: Python 3 표준 라이브러리만 사용(훅 스크립트), JSON 설정
- 배포: 팀 공용 템플릿 저장소 또는 Claude Code 플러그인으로 묶어 저장소마다 설치
- 외부 연동: 없음

## 구현 단계 (MVP → 확장)

1. MVP: 설정 파일 + 훅 3개 + 지시문. 실제 작업 몇 개로 멈춤 없이 끝나는지 확인한다.
2. 결정 로그를 작업 끝에 요약 보고서로 만들어 주는 기능.
3. 플러그인으로 묶어 설치를 한 줄로.

## 열린 질문 / 리스크

- 모든 권한을 허용하므로, 저장소 안 문서나 웹 페이지에 심어진 프롬프트 인젝션이 시키는 명령도 실행된다. 격리된 환경과 중요 접속 정보 제거가 유일한 방어선이다.
- 권장안이 항상 맞지는 않다. 결정 로그를 반드시 검토한다.
- 훅 이벤트 이름, 입력 JSON 필드(`tool_input.questions`, `transcript_path`, `stop_hook_active`), 출력 형식은 Claude Code 버전에 따라 바뀔 수 있다. 구현 시점의 공식 문서로 확인하고, 질문 창 처리(3번)는 실제로 한 번 띄워 동작을 확인한다.
- 비대화형(`-p`) 실행에서는 `PermissionRequest` 훅이 호출되지 않는다는 보고가 있다. 이 경우에도 bypassPermissions 모드가 권한을 처리하므로 문제는 없다.
- Stop 훅의 질문 판별이 정규식이라 놓치거나 잘못 잡을 수 있다. `stop_hook_active`로 연속 1회만 계속시켜 무한 반복은 막는다.

## 요구사항 후보

| ID | 제목 | 설명 | 우선순위 | 상태 | 출처 |
|---|---|---|---|---|---|
| REQ-후보-01 | 훅 기반 판정 | 권한 요청은 훅으로 받아 allow / deny / ask와 이유를 반환한다 | 상 | 폐기 | 결정 사항 (전부 허용으로 변경, REQ-후보-09로 대체) |
| REQ-후보-02 | 규칙 우선 | 결정적 규칙을 LLM 판정보다 먼저 적용하고, 거부 규칙은 뒤집지 않는다 | 상 | 폐기 | 결정 사항 (요청별 판정 안 함) |
| REQ-후보-03 | 실패 시 사람에게 | 판정기 오류·시간 초과는 ask로 처리한다 | 상 | 폐기 | 결정 사항 (사람 개입 없음) |
| REQ-후보-04 | 판정기 입력 격리 | LLM 판정기에는 요청 내용만 주고 대화·파일 내용은 주지 않는다 | 상 | 폐기 | 결정 사항 (LLM 판정기 없음) |
| REQ-후보-05 | 셸 명령 분해 | 이어진 셸 명령은 쪼개서 각각 판정하고 하나라도 거부면 거부한다 | 상 | 폐기 | 결정 사항 (요청별 판정 안 함) |
| REQ-후보-06 | 감사 로그 | 모든 판정을 요청 내용·결과·근거와 함께 기록한다 | 중 | 폐기 | 결정 사항 (REQ-후보-13으로 대체) |
| REQ-후보-07 | 원격 승인 | ask 요청을 메신저로 승인·거부하고 시간 초과 시 거부한다 | 중 | 폐기 | 결정 사항 (Slack 등 메신저 사용 불가) |
| REQ-후보-08 | 규칙 후보 제안 | 반복 승인 패턴을 "항상 허용" 규칙 후보로 제안하고 사람 승인 후 반영한다 | 하 | 폐기 | 결정 사항 (전부 허용이라 규칙 불필요) |
| REQ-후보-09 | 권한 전부 허용 | bypassPermissions 모드와 PermissionRequest 허용 훅으로 모든 권한 요청을 허용한다 | 상 | 승인 | 결정 사항 |
| REQ-후보-10 | 질문 창 권장안 응답 | AskUserQuestion은 권장안(없으면 첫 번째 선택지)을 고른 것으로 처리한다 | 상 | 승인 | 결정 사항 |
| REQ-후보-11 | 텍스트 질문 계속 진행 | 응답이 질문으로 끝나면 Stop 훅이 연속 1회까지 권장안으로 계속 진행시킨다 | 중 | 제안 | 구현 방법 4 |
| REQ-후보-12 | 무인 실행 지시문 | CLAUDE.md에 "질문하지 말고 권장안으로 진행" 지시문을 둔다 | 중 | 제안 | 구현 방법 5 |
| REQ-후보-13 | 결정 로그 | 허용한 요청과 자동으로 고른 답을 파일로 남기고 작업 끝에 보고한다 | 중 | 제안 | 구현 방법 2, 3, 5 |
| REQ-후보-14 | 격리 환경 전용 | 컨테이너·VM 등 운영 접속 정보가 없는 환경에서만 사용한다 | 상 | 제안 | 구현 방법 6 |

## 참고 자료

- Claude Code 훅 문서: https://code.claude.com/docs/en/hooks
- Claude Code 권한·설정 문서: https://code.claude.com/docs/en/settings
- 참고 이슈: `PermissionRequest` 훅은 비대화형(`-p`) 모드에서 실행되지 않는다는 지적 — https://github.com/anthropics/claude-code/issues/26703
- 관련 오픈소스 (2026-11 조사, 코드·유지보수 상태는 미검증. 대부분 요청별 판정 방식이라 이번 결정과는 다르지만 훅 작성 예시로 참고):
  - psd401/claude-auto-approve (규칙 기반 자동 허용, JSONL 로그) — https://github.com/psd401/claude-auto-approve
  - froggeric/claude-smart-approval (AI 판정 + 승인 패턴 학습) — https://github.com/froggeric/claude-smart-approval
  - ahmed-anas/claude-gatekeeper (Haiku로 판정) — https://github.com/ahmed-anas/claude-gatekeeper
  - malcomsonbrothers/claude-code-permission-hook (AI 판정 + 캐시, npm 설치) — https://github.com/malcomsonbrothers/claude-code-permission-hook
  - liberzon/claude-hooks (복합 명령 분해 검사) — https://github.com/liberzon/claude-hooks
  - karanb192/claude-code-hooks (위험 명령 차단 모음) — https://github.com/karanb192/claude-code-hooks
  - yigitkonur/hooks-claude-code (계획 승인 자동 허용) — https://github.com/yigitkonur/hooks-claude-code
  - yuuichieguchi/claude-remote-approver, Manta-Network/cc-remote-approval, jeremylongshore/claude-code-slack-channel (원격 승인 — 이번 결정에서는 사용 안 함)
