# Mr. Daebak Dinner

"특별한 날을 더욱 특별하게" — 웹 화면과 AI 챗봇 대화로 디너를 주문하고, 주방·배송 담당 직원이 주문 상태를 관리하는 디너 배달 주문 관리 시스템.

- 개발 기간: `2025.09 ~ 2025.12`
- 나의 역할: `프론트엔드, AI, 배포`

## 핵심 포인트

- AI 챗봇 주문: 자연어 대화만으로 메뉴, 스타일, 예약 시간, 배달지, 결제 정보를 추출해 주문을 생성합니다.
- 디너 주문: VALENTINE, FRENCH, ENGLISH, CHAMPAGNE 4종 디너와 SIMPLE, GRAND, DELUXE 3종 스타일 조합을 지원합니다.
- 주문 상태 관리: ORDERED, COOKING, COOKED, DELIVERING, DELIVERED 흐름에 따라 Chef·Delivery 화면에서 상태를 갱신합니다.
- 재고 연동: 주문 시 재고를 확인하고, 부족하면 주문을 차단합니다.
- 역할 기반 접근 제어: Interceptor로 고객/직원/주문 소유자 여부를 검증합니다.

## 기술 스택

- Java 17, Spring Boot 3.5.5 (Web, Data JPA, Thymeleaf, Validation)
- PostgreSQL, Gradle
- Ollama(gemma3:12b) 기반 AI 챗봇, JSON Schema 기반 구조화 출력
- Docker, Docker Compose

## 프로젝트 구조 (요약)

```
src/main/java/com/devak/mrdaebakdinner/
├── controller/     # CustomerController, StaffController, OrderController, AiOrderController
├── service/        # CustomerService, StaffService, OrderService, InventoryService, AiOrderService
├── repository/     # Spring Data JPA Repository
├── entity/         # OrderEntity, CustomerEntity, InventoryEntity, ItemEntity ...
├── dto/            # 화면·세션·API 전달용 DTO
├── mapper/         # Entity - DTO 변환
├── interceptor/    # 로그인/권한 검증
└── exception/      # 커스텀 예외, GlobalExceptionHandler

src/main/resources/
├── application.properties
├── data.sql        # 초기 품목/재고 데이터
├── static/         # CSS, JS, 이미지
└── templates/      # Thymeleaf 뷰 (customer/, staff/)
```

## 담당 기여 파트 — AI 챗봇 주문 기능

`AiOrderService`, `AiOrderController`에서 자연어 대화로 디너를 주문하는 기능을 담당했습니다.

- 멀티턴 대화 상태 관리: 사용자별 대화 이력을 유지하면서 메뉴, 스타일, 예약 시간, 배달지, 결제 정보를 여러 턴에 걸쳐 수집하고, 이미 확보한 정보가 유실되지 않도록 상태를 누적 관리했습니다.
- JSON Schema 기반 구조화 출력: Ollama의 format 옵션에 JSON Schema를 정의해 응답을 status, message, extracted_info 형태로 강제함으로써 자유 텍스트 파싱 오류를 줄였습니다.
- 프롬프트 엔지니어링: 디너별 기본 구성 품목 자동 로드, 유사 품목(빵/바게트, 커피잔/커피포트, 샴페인/와인) 혼동 방지, 상대적 시간 표현의 절대 시각 변환, 챔페인 디너 스타일 제약 등 도메인 규칙을 System Prompt로 강제했습니다.
- 2단계 응답 설계: 대화 진행 중(CONTINUE)에는 다음 질문만 반환하고, 필수 정보가 모두 채워지면(DONE) 별도의 엄격한 Schema로 최종 주문 JSON을 재추출해 신뢰도를 높였습니다.
- 트러블슈팅: 로컬 Ollama에서 Ollama Cloud로 전환, 시간/날짜 파싱 오류, 다중 아이템 동시 요청 시 수량 매칭 오류 등을 디버깅하며 프롬프트와 파싱 로직을 반복 개선했습니다.

관련 파일: `src/main/java/com/devak/mrdaebakdinner/service/AiOrderService.java`, `src/main/java/com/devak/mrdaebakdinner/controller/AiOrderController.java`

## 빠른 시작

### 1) 로컬 개발

PostgreSQL을 준비하고 환경 변수를 설정한 뒤:

```bash
./gradlew bootRun
```

### 2) Docker (로컬 통합)

```bash
docker compose up --build
```

`http://localhost:8080` 에서 접속합니다.

### 3) 테스트

```bash
./gradlew test
```

## 환경 변수 (주요)

- POSTGRES_DB
- POSTGRES_USER
- POSTGRES_PASSWORD
- SPRING_DATASOURCE_URL
- SPRING_DATASOURCE_USERNAME
- SPRING_DATASOURCE_PASSWORD
- OPENAI_API_KEY
- OLLAMA_API_BASEURL

프로젝트 루트의 `.env` 파일을 사용합니다 (`.gitignore`에 등록되어 Git에는 커밋되지 않습니다).
