# 인프라와 CI/CD

## 운영 구성

원페이지의 웹 애플리케이션, 서비스 API, AI 분석 서비스는 독립적으로 배포할 수 있는 컨테이너 서비스로 운영됩니다.

| 영역 | 사용 기술 |
| --- | --- |
| 실행 환경 | Google Cloud Run |
| 관계형 데이터 | Cloud SQL for PostgreSQL |
| 파일 자원 | Google Cloud Storage |
| AI 생성 | Google Gemini API |
| 자동 배포 | GitHub Actions |

## 분리 배포를 선택한 이유

- 사용자 화면 변경과 분석 처리 변경을 독립적으로 배포할 수 있습니다.
- 특정 영역에서 발생한 문제를 역할별로 추적하기 쉽습니다.
- 분석처럼 처리 시간이 긴 기능을 별도 서비스 책임으로 다룰 수 있습니다.

## 배포 흐름

```mermaid
flowchart LR
    A[변경 검증] --> B[GitHub Actions]
    B --> C[Container Build]
    C --> D[Cloud Run Deployment]
    D --> E[서비스 동작 확인]
```

## 문서상 제외 항목

운영 안전을 위해 클라우드 프로젝트 식별자, 서비스 계정, 실제 환경변수, 배포 URL, 데이터베이스 연결 정보, 로그 원문은 공개하지 않습니다.
