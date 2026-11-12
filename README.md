# sik-ai-idea

AI로 구체화하기 위한 아이디어 모음. 각 아이디어는 `카테고리/아이디어/README.md` 한 파일로 정리되어 있어, 구현은 별도 리포지터리에서 이 문서들을 참조해 진행한다.

- 작성 규칙: [CLAUDE.md](CLAUDE.md)
- 새 아이디어 양식: [TEMPLATE.md](TEMPLATE.md)

## 아이디어 목록

<!-- 카테고리와 아이디어는 각각 폴더 이름순으로 정렬한다. -->
<!-- 형식:
### [카테고리명](카테고리/)
- [아이디어 제목](카테고리/아이디어/README.md) — 한 줄 요약
-->

### [ai-dev-tools](ai-dev-tools/)
- [Claude Code 권한 요청 자동 판정 에이전트](ai-dev-tools/permission-auto-approver/README.md) — 훅으로 권한 요청을 받아 정책에 따라 자동 허용·거부, 애매한 것만 사람에게 원격 승인

### [compliance](compliance/)
- [계정·권한 정기 검토 보조](compliance/access-review-assistant/README.md) — 인사 데이터와 시스템 계정을 대조해 이상 계정을 찾고 부서장 확인까지 자동화
- [ISMS-P 증적 자동 수집·매핑](compliance/isms-p-evidence-collector/README.md) — 통제항목별 증적을 자동 수집·매핑하고 누락·만료 항목 알림
- [법령 변경 추적 및 사내 규정 신구대조표 자동 작성](compliance/law-change-tracker/README.md) — 법령정보 API로 개정을 감지해 사내 규정 개정안과 신구대조표 생성
- [개인정보 처리방침-실제 수집 정합성 점검](compliance/privacy-policy-consistency-checker/README.md) — 처리방침과 실제 폼·쿠키·외부 스크립트를 비교해 고지 누락 탐지
- [보안성 검토 1차 리뷰어](compliance/security-review-assistant/README.md) — 도입·변경 요청 문서를 체크리스트로 1차 검토하고 질문 목록 생성

### [game](game/)
- **[공통 지침]** [게임 개발·출시 공통 지침](game/dev-release-guideline/README.md) — 엔진(Unity)과 코드 구조, 서버, 다국어(1차 4개 언어), 개발 도구와 AI 사용, 아트 파이프라인, 출시 체크리스트, 개발 단계
- **[공통 지침]** [게임 개발 공통 과금 지침](game/monetization-guideline/README.md) — 단일 재화, 상위 상품 효율 상승, 등급 없는 뽑기, 광고와 광고 제거 상품, 환불 규정, 스팀판 과금, 무과금 클리어 보장
- **[공통 지침]** [스테이지 기반 전략 게임 기본 골격](game/stage-strategy-framework/README.md) — 게임 규칙: 가로 쿼터뷰 RPG(넓은 필드·액션 + 필수 턴제 + 보스룸), 캐릭터·장비·성장, 화면과 대화 UI, 플랫폼 확장 대비
- [아틀란티스 그리드 RPG (가칭)](game/atlantis-grid/README.md) — 아틀란티스 테마: 가라앉기 직전의 해양 문명을 지키며 수중 도시와 심해를 탐험한다
- [코스믹 호러 그리드 RPG (가칭)](game/cosmic-horror-grid/README.md) — 코스믹 호러 테마: 1920년대 조사원들이 사교 집단을 막고 깨어나는 고대 존재를 다시 봉인한다 (H. P. 러브크래프트 원전)
- [이집트 신화 그리드 RPG (가칭)](game/egypt-myth-grid/README.md) — 이집트 신화 테마: 호루스·아누비스·바스테트가 세트와 혼돈의 뱀에 맞서 질서를 되찾는다
- [판타지 그리드 RPG (가칭)](game/fantasy-grid/README.md) — 판타지 세계 테마: 인간·엘프·드워프 모험가가 마왕군에 맞서 봉인석을 되찾는다
- [조선 그리드 RPG (가칭)](game/joseon-grid/README.md) — 조선 500년 테마: 건국부터 개항까지 장마다 시대가 바뀌고, 이성계·이순신 등 조선의 인물과 함께 싸운다
- [원탁 그리드 RPG (가칭)](game/medieval-europe-grid/README.md) — 중세 유럽 테마: 아서왕 전설의 원탁 기사들과 성을 공략하고 기사 결투를 치른다
- [북유럽 신화 그리드 RPG (가칭)](game/norse-myth-grid/README.md) — 북유럽 신화 테마: 토르·발키리·오딘과 아홉 세계를 오가며 신들의 황혼에 맞선다
- [SF 그리드 RPG (가칭)](game/space-war-grid/README.md) — SF 우주전쟁 테마: 독립 함대를 이끌고 제국과 싸우며, 함대전은 턴제, 에이스 대결은 1:1 액션
- [삼국지 그리드 RPG](game/three-kingdoms-grid-tactics/README.md) — 삼국지 테마: 유비·관우·장비와 필드를 모험하고 연의의 핵심 전투는 턴제, 보스룸은 부하·챔피언을 포함한 액션 (오리지널·미녀 재해석 디자인 선택)
- [와일드 웨스트 그리드 RPG (가칭)](game/wild-west-grid/README.md) — 서부시대 테마: 보안관과 총잡이들이 무법자 소굴을 소탕하고 정오의 결투를 벌인다
- [무협 그리드 RPG (가칭)](game/wuxia-grid/README.md) — 무협 테마: 멸문 문파의 제자가 강호를 누비며 경공으로 탐험하고 비무를 치른다 (창작 문파·무공)
- [좀비 그리드 RPG (가칭)](game/zombie-apocalypse-grid/README.md) — 좀비 아포칼립스 테마: 생존자 팀이 무너진 도시를 탈출해 백신 연구소로 향한다

### [security-awareness](security-awareness/)
- [부서 맞춤형 모의 훈련 시나리오 생성](security-awareness/phishing-simulation-generator/README.md) — 부서 업무 맥락에 맞는 모의 피싱 시나리오와 교육 자료 생성
- [사내 보안 정책 Q&A 봇](security-awareness/security-policy-qa-bot/README.md) — 정책 조항을 인용해 반복 질문에 답하고 미해결 질문은 티켓으로

### [security-ops](security-ops/)
- [보안 관제 경보 1차 분류(트리아지)](security-ops/alert-triage-assistant/README.md) — 경보 맥락을 자동 수집해 오탐 가능성과 권장 조치 요약
- [피싱 신고 메일 자동 분석](security-ops/phishing-report-analyzer/README.md) — 신고 메일 판정, 동일 수신자 일괄 조치, 신고자 자동 회신
- [자산 맞춤형 위협 인텔리전스 브리핑](security-ops/threat-intel-briefing/README.md) — 보유 자산과 관련된 CVE·보안 공지만 골라 매일 요약
- [취약점 진단 결과 정규화 및 조치 가이드 생성](security-ops/vuln-report-normalizer/README.md) — 스캐너·모의해킹 결과 통합, 중복 제거, 개발자용 수정 가이드

### [security-solution](security-solution/)
- [AI·LLM 보안](security-solution/ai-llm-security/README.md) — (신규) 외부 AI 사용 시 민감정보 마스킹, 사내 LLM 입출력 검사, 자동 레드팀
- [백신·EDR 엔진](security-solution/antivirus-edr-engine/README.md) — (사내 개발) 샘플 분석 보고서와 YARA·행위 룰 초안 자동 생성, 오탐 측정 후 배포
- [인증·PKI·MFA](security-solution/auth-pki-mfa/README.md) — (신규) SSO·MFA·사설 CA, 레거시 연동 커넥터와 표준 적합성 테스트 자동 생성
- [클라우드 워크로드 보호 (CWPP)](security-solution/cloud-workload-protection/README.md) — (신규) 이미지·설정 점검 정책과 eBPF 런타임 탐지 룰을 기준 문서에서 자동 생성
- [DB·서버 접근통제](security-solution/db-server-access-control/README.md) — (사내 개발) DBMS 프로토콜 파서 생성, 버전 매트릭스 회귀 테스트, 자연어 위험 명령 정책
- [DRM 문서보안](security-solution/drm-document-security/README.md) — (사내 개발) 응용프로그램 버전별 호환성 테스트 팜과 장애 분석 보조
- [침입탐지·방지 시스템 (IDS/IPS)](security-solution/ids-ips/README.md) — (사내 개발) CVE·PoC로 탐지 룰과 검증 트래픽을 생성하고 회귀 테스트
- [모바일 앱 보안](security-solution/mobile-app-security/README.md) — (신규) 앱 위변조·루팅·후킹 탐지, 우회 기법 재현 테스트, 취약점 점검 보고서
- [네트워크 접근 통제 (NAC)](security-solution/nac/README.md) — (사내 개발) 장비 연동 어댑터 자동 생성, 단말 자동 식별, 정책 시뮬레이션
- [망연계 솔루션](security-solution/network-separation-transfer/README.md) — (사내 개발) 연계 어댑터와 CDR 무해화 모듈 자동 생성, 데이터 등급별 연계 정책
- [OT·IoT 보안](security-solution/ot-iot-security/README.md) — (신규) 산업용 프로토콜 파서 자동 생성, 수동 방식 자산 식별과 통신 기준선 탐지
- [개인정보 검색](security-solution/pii-discovery/README.md) — (사내 개발) 패턴 검출 + 문맥 판별로 오탐 감소, 문서 파서·합성 테스트 셋 자동 생성
- [SIEM·통합로그](security-solution/siem-log-integration/README.md) — (신규) 로그 샘플로 파서 자동 생성, 상관분석 룰 백테스트, 자연어 검색
- [취약점 진단 자동화](security-solution/vuln-assessment-automation/README.md) — (신규) 국내 진단 기준 점검 스크립트를 자동 생성하고 양호·취약 환경에서 검증
- [웹방화벽 (WAF)](security-solution/waf/README.md) — (신규) 우회 변형 공격 자동 테스트, 학습 기반 허용 정책, 오탐 예외 초안
