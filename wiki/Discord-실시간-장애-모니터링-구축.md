# [트러블슈팅] Discord 실시간 에러 알림 및 서버 상태 리포트 시스템 구축

## 1. 문제 발생 배경
운영 서버(`1-page.site`) 배포 후, 간헐적으로 발생하는 AI 분석 실패나 API 타임아웃 오류를 즉시 인지하기 어려웠습니다. 사용자가 직접 제보하기 전까지는 에러 발생 사실을 알 수 없어 대응이 늦어지는 문제가 반복되었습니다.

## 2. 문제 정의
운영 중 발생하는 심각한 오류(ERROR, CRITICAL)를 실시간으로 탐지하고, 개발팀이 상주하는 **Discord** 채널로 즉시 전파하는 모니터링 파이프라인 구축이 필요했습니다. 또한, 주기적인 서버 상태 요약을 통해 잠재적인 위협을 선제적으로 파악하고자 했습니다.

## 3. 기술적 선택 및 파이프라인 설계

### 3.1. 전체 아키텍처
1. **GCP Log Router Sink**: Cloud Run(`fe`, `be`, `ai`)에서 발생하는 `severity >= ERROR` 로그를 필터링하여 Pub/Sub으로 전달.
2. **GCP Pub/Sub**: 로그 데이터를 비동기적으로 메시지화하여 전송.
3. **Cloud Function (Gen2)**: Pub/Sub 트리거를 받아 로그를 분석하고 Discord Webhook으로 전송.

### 3.2. 실시간 알림 (`onepage-error-notifier`)
- **자동 원인 분석**: 로그 내 키워드(`ECONNRESET`, `TimeoutError`, `403` 등)를 매칭하여 10종의 대표 원인으로 자동 분류.
- **상세 정보 포함**: 발생 시각(KST), 서비스명, 원본 로그, 그리고 즉시 확인 가능한 GCP Console 로그 링크를 함께 발송.

### 3.3. 정기 상태 리포트 (`onepage-status-reporter`)
- **주기적 집계**: 6시간마다(하루 4회) 최근 발생한 에러 건수를 요약 보고.
- **Cloud Scheduler 연동**: HTTP 트리거를 통해 정해진 시간에 리포트 생성 함수 호출.

## 4. 구현 과정 중 트러블슈팅

### 4.1. Cloud Function 권한 이슈
- **증상**: `PERMISSION_DENIED: logging.logEntries.list` 발생.
- **해결**: 서비스 계정(`onepage-app-sa`)에 `roles/logging.viewer` 및 `roles/run.invoker` 권한을 부여하여 해결.

### 4.2. 함수 시그니처 오류
- **증상**: `TypeError: notify_discord() takes 1 positional argument but 2 were given`.
- **원인**: Gen2 Pub/Sub 트리거 시 인자 전달 방식의 변화.
- **해결**: `@functions_framework.cloud_event` 데코레이터를 적용하여 표준 인터페이스 준수.

## 5. 개선 결과 및 회고
- **장애 인지 속도**: 평균 **10초 이내**에 장애 상황을 Discord를 통해 확인 가능.
- **분석 효율**: 자동 분류 기능을 통해 에러 발생 시 로그를 직접 뒤지기 전에도 대략적인 원인(DB 문제, API 권한 등)을 파악할 수 있게 됨.
- **안정성**: 6시간 단위 리포트를 통해 빈도가 낮은 간헐적 에러 패턴도 놓치지 않고 트래킹 가능.

이번 모니터링 시스템 구축은 '만드는 것'만큼 '운영하는 것'이 중요하다는 것을 일깨워준 중요한 과정이었습니다.
