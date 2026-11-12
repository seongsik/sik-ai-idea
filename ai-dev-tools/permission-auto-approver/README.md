# Claude Code 권한 요청 자동 판정 에이전트

> 한 줄 요약: Claude Code가 도구 실행 권한을 물을 때마다 사람이 "허용/항상 허용"을 누르는 대신, 사내 정책에 따라 안전한 요청은 자동 허용하고 위험한 요청은 거부하거나 사람에게 넘기는 판정기를 만든다.

- 카테고리: ai-dev-tools
- 상태: 아이디어
- 작성일: 2026-11-12
- 구현 리포지터리: -

## 배경 / 문제

- Claude Code로 긴 작업을 맡기면 `npm test`, `git status` 같은 뻔한 명령에도 권한 확인 창이 떠서 작업이 멈추고, 사람이 화면 앞에 붙어 있어야 한다.
- 그렇다고 모든 권한을 건너뛰는 모드(`--dangerously-skip-permissions`, bypassPermissions)를 쓰면 `rm -rf`, 비밀키 읽기, 운영 서버 접속까지 막을 수 없다.
- 개발자마다 "항상 허용" 규칙을 제각각 쌓아 두므로 팀 차원의 기준과 감사 기록이 없다.

## 구현 방식 비교 (결론)

| 방식 | 동작 | 판단 |
|---|---|---|
| A. 훅 기반 판정기 (권장) | Claude Code의 `PermissionRequest`·`PreToolUse` 훅이 요청 내용(도구 이름, 명령, 파일 경로)을 JSON으로 받아 allow / deny / ask를 돌려준다 | 공식 기능, 요청 내용을 정확히 볼 수 있음, 빠르고 기록이 남음 |
| B. 권한 규칙 자동 관리 | 실제 승인 이력을 분석해 `settings.json`의 `permissions.allow`·`deny` 규칙 후보를 만들고 사람이 승인 | "항상 허용"을 체계적으로 대체. A와 함께 쓴다 |
| C. Agent SDK `canUseTool` 콜백 | SDK로 Claude를 직접 돌리는 경우 콜백 함수에서 판정 | 자체 에이전트 서비스를 만들 때 사용 |
| D. 화면 입력 대행 (컴퓨터 사용 모델, 오픈소스 GUI 에이전트, xdotool·pyautogui) | 화면을 읽고 "허용" 버튼을 대신 누른다 | 비권장. 화면 문구로만 판단해 무엇을 승인하는지 정확히 모르고, UI가 바뀌면 깨지며, 결국 "무조건 허용"과 같아진다. 훅을 쓸 수 없는 데스크톱 GUI 앱에만 최후 수단 |

## 목표

- 정책상 안전한 요청(읽기, 테스트, 린트, 작업 폴더 안 편집)은 사람 개입 없이 자동 허용한다.
- 정책상 금지된 요청(작업 폴더 밖 삭제, 비밀 파일 읽기, 강제 푸시, 운영 접속)은 자동 거부하고 이유를 Claude에게 돌려준다.
- 애매한 요청만 사람에게 묻고, 메신저·모바일로 승인할 수 있게 한다.
- 모든 판정을 기록하고, 반복 승인된 패턴을 "항상 허용" 규칙 후보로 제안한다.

## 범위 밖 (Non-goals)

- 권한 확인을 통째로 끄는 것 (판정 없이 전부 허용).
- 화면 클릭 자동화를 기본 방식으로 쓰는 것 (구현 방식 비교 D).
- Claude Code 자체 수정.

## 사용자 / 사용 시나리오

1. 개발자가 팀 공용 판정기 훅을 `.claude/settings.json`에 등록하고 Claude Code에 긴 작업을 맡긴다.
2. Claude가 `npm test`를 실행하려 하면 훅이 허용 규칙에 맞는 것을 확인하고 바로 허용한다.
3. Claude가 `.env`를 읽으려 하면 훅이 거부하고 "비밀 파일은 읽을 수 없음, 예시 파일을 참고할 것"이라는 이유를 돌려준다. Claude는 다른 방법으로 작업을 이어 간다.
4. 규칙에 없는 `curl ... | sh` 요청은 LLM 판정기가 위험도를 매기고, 중간 이상이면 개발자 메신저로 승인 요청을 보낸다.
5. 주간 보고서에 "사람이 10번 넘게 허용한 패턴"이 규칙 후보로 올라오고, 팀 리드가 승인하면 공용 규칙에 반영된다.

## 핵심 기능

- [ ] 훅 판정기: 요청 JSON을 받아 allow / deny / ask와 이유를 반환
- [ ] 정책 3단계: 결정적 규칙(허용·거부 목록, 경로·명령 패턴) → LLM 위험도 판정 → 사람 승인
- [ ] 명령 분석: 파이프·`&&`·서브셸로 이어진 셸 명령을 쪼개 각각 판정
- [ ] 원격 승인: 메신저·모바일로 ask 요청 승인/거부, 시간 초과 시 거부
- [ ] 판정 감사 로그와 대시보드
- [ ] 승인 이력 기반 "항상 허용" 규칙 후보 제안

## 기술 구상

- 기술 스택: Python 또는 TypeScript(훅 스크립트), 셸 명령 파서(bashlex 등), 정책 파일(YAML), 판정용 LLM(Claude API 소형 모델 또는 사내 모델), SQLite/PostgreSQL(로그)
- 아키텍처: Claude Code → 훅(`PermissionRequest`/`PreToolUse`) → 판정기(규칙 → LLM → 사람) → 결과 JSON → Claude Code / 로그 → 규칙 제안 배치
- 외부 연동: Slack·Teams(원격 승인), SIEM(감사 로그), 사내 SSO(승인자 확인)
- 판정 원칙:
  - 결정적 규칙이 LLM보다 먼저 적용되고, 거부 규칙은 어떤 판정으로도 뒤집히지 않는다.
  - LLM 판정기에는 요청 내용만 주고, 작업 중인 대화나 파일 내용은 주지 않는다 (프롬프트 인젝션이 판정기로 번지는 것을 막는다).
  - 판정기 오류·시간 초과는 ask(사람에게 넘김)로 처리한다.

## 구현 단계 (MVP → 확장)

1. MVP: 결정적 규칙만 쓰는 훅 판정기 + 로그. 허용·거부 목록 기본값 제공.
2. LLM 위험도 판정, 셸 명령 분해, 메신저 원격 승인.
3. 승인 이력 기반 규칙 제안, 팀 공용 정책 배포, Agent SDK `canUseTool` 어댑터.
4. (선택) 훅을 쓸 수 없는 GUI 도구용 화면 입력 대행 어댑터. 이때도 판정은 같은 정책 엔진을 거친다.

## 열린 질문 / 리스크

- 자동 허용은 보안 통제를 줄이는 일이다. 사내 보안 정책상 허용 가능한 자동 승인 범위를 먼저 정해야 한다.
- 저장소 안의 문서·코드에 심어진 프롬프트 인젝션이 Claude에게 위험 명령을 시킬 수 있다. 판정기가 마지막 방어선이므로 거부 규칙을 보수적으로 둔다.
- 훅 이벤트 이름과 입출력 형식은 Claude Code 버전에 따라 바뀔 수 있다. 구현 시점의 공식 문서를 기준으로 한다.
- Claude Code에는 AI 분류기가 권한을 판정하는 auto 모드와 공식 프롬프트 기반(LLM) 훅이 이미 있고, 같은 목적의 오픈소스 훅도 여럿 있다(참고 자료). 새로 만들기보다 이것들을 조합하고 사내 정책·원격 승인·감사 기록 부분에 집중할지 정한다.
- `PermissionRequest` 훅은 비대화형(`-p`) 실행에서는 호출되지 않는다는 보고가 있다. 무인 실행에서는 `PreToolUse` 훅이나 SDK `canUseTool`로 판정한다.
- 화면 입력 대행 방식은 무엇을 승인하는지 확인할 수 없어 감사 요건을 만족하기 어렵다.

## 요구사항 후보

| ID | 제목 | 설명 | 우선순위 | 상태 | 출처 |
|---|---|---|---|---|---|
| REQ-후보-01 | 훅 기반 판정 | 권한 요청은 훅으로 받아 allow / deny / ask와 이유를 반환한다 | 상 | 제안 | 구현 방식 비교 |
| REQ-후보-02 | 규칙 우선 | 결정적 규칙을 LLM 판정보다 먼저 적용하고, 거부 규칙은 뒤집지 않는다 | 상 | 제안 | 기술 구상 |
| REQ-후보-03 | 실패 시 사람에게 | 판정기 오류·시간 초과는 ask로 처리한다 | 상 | 제안 | 기술 구상 |
| REQ-후보-04 | 판정기 입력 격리 | LLM 판정기에는 요청 내용만 주고 대화·파일 내용은 주지 않는다 | 상 | 제안 | 기술 구상 |
| REQ-후보-05 | 셸 명령 분해 | 이어진 셸 명령은 쪼개서 각각 판정하고 하나라도 거부면 거부한다 | 상 | 제안 | 핵심 기능 |
| REQ-후보-06 | 감사 로그 | 모든 판정을 요청 내용·결과·근거와 함께 기록한다 | 중 | 제안 | 핵심 기능 |
| REQ-후보-07 | 원격 승인 | ask 요청을 메신저로 승인·거부하고 시간 초과 시 거부한다 | 중 | 제안 | 핵심 기능 |
| REQ-후보-08 | 규칙 후보 제안 | 반복 승인 패턴을 "항상 허용" 규칙 후보로 제안하고 사람 승인 후 반영한다 | 하 | 제안 | 핵심 기능 |

## 참고 자료

- Claude Code 훅 문서: https://code.claude.com/docs/en/hooks
- Claude Code 권한·설정 문서: https://code.claude.com/docs/en/settings
- Claude Agent SDK 권한 처리(`canUseTool`) 문서
- 관련 오픈소스 (2026-11 조사, 코드·유지보수 상태는 미검증):
  - 규칙 기반 자동 허용: psd401/claude-auto-approve (설정의 허용 패턴을 쓰고 deny/ask 규칙 존중, JSONL 로그) — https://github.com/psd401/claude-auto-approve
  - AI 판정 + 학습: froggeric/claude-smart-approval (모르는 Bash 명령은 AI 판정, 복합 명령에서도 거부 목록 우선, 승인 패턴 학습) — https://github.com/froggeric/claude-smart-approval
  - AI 판정: ahmed-anas/claude-gatekeeper (Haiku로 판정, 위험·애매하면 이유와 함께 거부) — https://github.com/ahmed-anas/claude-gatekeeper
  - AI 판정 + 캐시: malcomsonbrothers/claude-code-permission-hook (npm 설치) — https://github.com/malcomsonbrothers/claude-code-permission-hook
  - 복합 명령 분해: liberzon/claude-hooks (PreToolUse에서 하위 명령별로 허용·거부 패턴 검사) — https://github.com/liberzon/claude-hooks
  - 위험 명령 차단 모음: karanb192/claude-code-hooks — https://github.com/karanb192/claude-code-hooks
  - 모바일 원격 승인: yuuichieguchi/claude-remote-approver (ntfy 푸시, 허용·항상 허용·거부 버튼) — https://github.com/yuuichieguchi/claude-remote-approver
  - 원격 승인 플러그인: Manta-Network/cc-remote-approval (로컬 창과 원격 중 먼저 답한 쪽 적용) — https://github.com/Manta-Network/cc-remote-approval
  - Slack 승인 + 정책 + 서명 감사 로그: jeremylongshore/claude-code-slack-channel — https://github.com/jeremylongshore/claude-code-slack-channel
- 참고 이슈: `PermissionRequest` 훅은 비대화형(`-p`) 모드에서 실행되지 않는다는 지적 — https://github.com/anthropics/claude-code/issues/26703
