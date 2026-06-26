# 전지민 (Jimin Jeon) — Backend Developer Portfolio

**데이터 직무를 거쳐 백엔드로 전환한, 영어 협업이 가능한 개발자입니다.**

미국 현지 기업에서 데이터 직무를 수행했고, 현재는 Spring 기반으로 결제 정합성과 부하 테스트 기반 성능 진단 등 정확하고 안정적으로 동작하는 서버에 집중하고 있습니다.

📫 **Email** · [jeemin065@gmail.com] &nbsp;|&nbsp; 💻 **GitHub** · [@jeemin65-pixel](https://github.com/jeemin65-pixel)

> 이 저장소는 부트캠프 팀 프로젝트 2건을 정리한 포트폴리오입니다.
> 각 프로젝트의 상세 코드는 원본 팀 저장소 링크에서 확인하실 수 있습니다.

---

## 🛠 기술 스택

| 구분 | 내용 |
|------|------|
| **Language** | Java 21, SQL (MySQL, PostgreSQL) |
| **Framework** | Spring Boot 3.x / 4.x, Spring Data JPA (JPQL) |
| **인증 / 결제** | OAuth 2.0 (Kakao, Google), Toss Payments |
| **AI / LLM** | OpenAI API |
| **Cloud (사용 경험)** | GCP Cloud Functions (2023, 데이터 변환 자동화), AWS S3 (백엔드 이미지 업로드 연동) |
| **Test / 부하** | JUnit, k6 |
| **협업** | Git, Jira |

---

## 📦 Projects

### 1. ShoppingFourU — 전자상거래 플랫폼

> 장바구니·주문·결제 도메인을 갖춘 e-커머스 백엔드. 대용량 트래픽 상황을 가정한
> 부하 테스트로 결제 동시성 병목을 진단하고 구조 개선 방안을 도출했습니다.

- **기간 / 인원**  [2025.10 ~ 2026.01] / 5인 팀 프로젝트
- **원본 저장소**  [kt-cloud-basic-project/shopping](https://github.com/kt-cloud-basic-project/shopping)
- **기술 스택**  Java 21, Spring Boot 3.5.7, JPA, MySQL, JUnit, OpenAI API, k6

**담당 역할**
- 장바구니 / 주문 도메인 설계 및 REST API 구현
- OpenAI API 기반 FAQ 챗봇 기능 적용 (시스템 프롬프트 설계, 벡터 스토어 연동 설정)
- k6 부하 테스트 설계 및 수행

**🔍 핵심 트러블슈팅 — 결제 동시성 데드락 진단**
- 대용량 트래픽을 가정한 k6 부하 테스트 중, 결제 동시 처리 시점에 응답이 멈추는 현상 확인
- 원인 분석: 이미 점유 중인 `@Transactional` 커넥션 안에서 `REQUIRES_NEW`로 새 트랜잭션을 요청하면서 **HikariCP 커넥션 풀 고갈 → 데드락** 발생
- 단순 풀 사이즈 증설이 아니라 **트랜잭션 경계 재설계**가 필요하다는 결론 도출
- 부하 테스트 결과: 「데이터셋 규모·동시 요청 수·p95 응답시간 등 실제 측정값 기입」

**🔧 AI 챗봇 통합 · 테스트 트러블슈팅**
- **테스트 격리로 `contextLoads` 에러 해결** — 외부 OpenAI API에 의존하는 `DefaultVectorApi`를 `FakeVectorApi`·`FakeChatApi`로 대체하고 `@Profile("!test")`를 적용해, ApplicationContext 로딩 검증이 외부 의존성 없이 통과하도록 개선 ([PR #160](https://github.com/kt-cloud-basic-project/shopping/pull/160))
- **DI 초기화 순서 버그 해결** — `OpenAIProperties`에서 파생되는 `token`이 `@RequiredArgsConstructor` 사용 시 의존성 주입보다 먼저 초기화되며 발생한 에러를, 수동 생성자로 초기화 순서를 명시해 해결 ([PR #160](https://github.com/kt-cloud-basic-project/shopping/pull/160))
- JUnit 기반 장바구니 / 주문 단위 테스트 작성으로 서비스 레이어 안정성 확보

---

### 2. MindLog AI — 심리상담 챗봇 서비스

> 상담 기록·크레딧·결제를 관리하는 백엔드. 결제 정합성과 크레딧 소진 로직 등
> "돈이 걸린 흐름"을 정확하게 처리하는 데 집중했습니다.

- **기간 / 인원**  [2026.01 ~ 2026.04] / 15인 팀 프로젝트
- **원본 저장소**  [8ocket/Backend](https://github.com/8ocket/Backend)
- **기술 스택**  Java 21, Spring Boot 4.0.1, JPA, PostgreSQL, OAuth 2.0, AWS S3, JUnit

**담당 역할**
- AI 상담 기록 도메인 설계 및 백엔드 개발 (사건 요약 · 감정 · 행동 제언 관리)
- 토스 페이먼츠 결제 연동
- 크레딧 시스템 구현 (유료 / 무료 크레딧 분리, 무료 크레딧 우선 소진)
- OAuth 2.0 기반 카카오 · 구글 소셜 로그인 구현
- AWS S3 연동 이미지 업로드 기능 구현

**🔍 핵심 포인트 — 결제 정합성 확보**
- 금액 위변조 방지: **주문 번호 · DB 저장 금액 · PG 승인 금액** 3중 대조 검증
- 중복 결제 방지 로직 구현으로 동일 결제 재요청 차단
- Fixture 기반 상담 요약 / 크레딧 도메인 테스트 코드 작성으로 핵심 로직 검증

**🔧 크레딧 정합성 트러블슈팅**
- **크레딧 잔액 중복 차감 버그 해결** —  잔액 합산 쿼리가 환불(REFUND) 거래까지 포함해 크레딧이 중복 차감되던 문제를 합산 대상에서 환불 거래를 제외(transactionType != 'REFUND')하도록 수정해 해결 ([PR #101](https://github.com/8ocket/Backend/pull/101))

**🔧 결제 검증 순서 리팩토링 — 검증 계층 분리 (PG 호출 전 / 후)**

- **문제 인식** — 기존 흐름은 토스 `confirm()`으로 결제를 **승인한 뒤** 금액·상태를 검증하는 구조. 위변조된 금액이나 중복 요청도 일단 PG 승인이 나간 뒤에야 걸러져, 잘못된 건은 별도의 결제 취소(보상 트랜잭션)가 필요해 흐름이 복잡해지고 정합성 리스크가 남음
- **개선 — 검증을 두 계층으로 분리**
  - **PG 호출 전 검증**: 결제 상태(`READY`)·`paymentKey` 중복·**요청 금액 vs DB 저장 금액**을 `tossClient.confirm()` 호출 이전에 수행 → 위변조·중복 결제는 승인 요청 자체가 나가지 않도록 차단
  - **PG 호출 후 검증**: 토스 응답이 있어야만 가능한 검증(`status == DONE`, `orderId` 일치, **DB 금액 vs 토스 승인 금액**)은 승인 후 수행
- **효과** — `요청 금액 == DB 금액 == 토스 승인 금액`이 체인으로 연결되어 금액 정합성을 끝까지 보장. 잘못된 요청은 PG 호출 전에 걸러지므로 불필요한 "승인 → 취소" 왕복이 제거됨 *(개인 리팩토링, 로컬 반영)*
---

## 👤 About

- **George Mason University**, Computational and Data Sciences 학사 (GPA 3.76 / 4.0)
- **KT Cloud Tech Up 부트캠프** (Backend Development 과정) 수료
- **데이터 직무 (약 2년 10개월)** · 공개 데이터 수집, 중복 데이터 제거 및 결측 데이터 보완, 제품 데이터 입력 및 네이밍 컨벤션 구축, GCP Cloud Function 기반 데이터 변환 자동화
- **SQL 과목 조교(TA)** · 과제 채점 및 멘토링
- English (Business Level)

<!--
체크리스트 (커밋 전 확인):
[ ] k6 부하 테스트 실측 수치 입력 (데이터셋 규모, 동시 요청 수, p95 등)
[ ] 데드락 원인 설명이 실제 분석 내용과 일치하는지 재확인
-->
