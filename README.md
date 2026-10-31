# sik-ai-idea

AI로 구체화하기 위한 아이디어 모음. 각 아이디어는 `카테고리/아이디어/README.md` 한 파일로 정리되어 있어, 그 파일만 가져가 다른 리포지터리에서 바로 구현을 시작할 수 있다.

- 작성 규칙: [CLAUDE.md](CLAUDE.md)
- 새 아이디어 양식: [TEMPLATE.md](TEMPLATE.md)

## 아이디어 목록

<!-- 카테고리와 아이디어는 각각 폴더 이름순으로 정렬한다. -->
<!-- 형식:
### [카테고리명](카테고리/)
- [아이디어 제목](카테고리/아이디어/README.md) — 한 줄 요약
-->

### [compliance](compliance/)
- [계정·권한 정기 검토 보조](compliance/access-review-assistant/README.md) — 인사 데이터와 시스템 계정을 대조해 이상 계정을 찾고 부서장 확인까지 자동화
- [ISMS-P 증적 자동 수집·매핑](compliance/isms-p-evidence-collector/README.md) — 통제항목별 증적을 자동 수집·매핑하고 누락·만료 항목 알림
- [법령 변경 추적 및 사내 규정 신구대조표 자동 작성](compliance/law-change-tracker/README.md) — 법령정보 API로 개정을 감지해 사내 규정 개정안과 신구대조표 생성
- [개인정보 처리방침-실제 수집 정합성 점검](compliance/privacy-policy-consistency-checker/README.md) — 처리방침과 실제 폼·쿠키·외부 스크립트를 비교해 고지 누락 탐지
- [보안성 검토 1차 리뷰어](compliance/security-review-assistant/README.md) — 도입·변경 요청 문서를 체크리스트로 1차 검토하고 질문 목록 생성

### [game](game/)
- [격자 턴제 삼국지 공성전](game/three-kingdoms-grid-tactics/README.md) — 기동력이 다른 장수로 격자 맵을 전진하며 제한 턴 안에 적 요새를 함락하는 전략 게임

### [security-awareness](security-awareness/)
- [부서 맞춤형 모의 훈련 시나리오 생성](security-awareness/phishing-simulation-generator/README.md) — 부서 업무 맥락에 맞는 모의 피싱 시나리오와 교육 자료 생성
- [사내 보안 정책 Q&A 봇](security-awareness/security-policy-qa-bot/README.md) — 정책 조항을 인용해 반복 질문에 답하고 미해결 질문은 티켓으로

### [security-ops](security-ops/)
- [보안 관제 경보 1차 분류(트리아지)](security-ops/alert-triage-assistant/README.md) — 경보 맥락을 자동 수집해 오탐 가능성과 권장 조치 요약
- [피싱 신고 메일 자동 분석](security-ops/phishing-report-analyzer/README.md) — 신고 메일 판정, 동일 수신자 일괄 조치, 신고자 자동 회신
- [자산 맞춤형 위협 인텔리전스 브리핑](security-ops/threat-intel-briefing/README.md) — 보유 자산과 관련된 CVE·보안 공지만 골라 매일 요약
- [취약점 진단 결과 정규화 및 조치 가이드 생성](security-ops/vuln-report-normalizer/README.md) — 스캐너·모의해킹 결과 통합, 중복 제거, 개발자용 수정 가이드
