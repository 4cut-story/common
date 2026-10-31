# common

서비스들이 함께 쓰는 빌드 설정과 계약의 원본입니다. 각 서비스는 이 레포의 **태그된 버전**에 고정해 씁니다.

| 경로 | 내용 |
|---|---|
| `parent-pom/` | Spring 서비스 공통 parent POM (Maven 공통 의존성, GitHub Packages 배포) |
| `contracts/` | 서비스별 OpenAPI, 공용 스키마, 게이트웨이 라우트 표 |
| `service-catalog.yaml` | 서비스 이름·포트·주소의 원본 |

## 계약 변경 순서

`common/contracts` PR → 태그 → 제공 서비스 구현 → 호출 서비스 업데이트. 하위 호환이 깨지면 `/v2` 경로로 올립니다.

설계와 스펙은 [docs](https://github.com/4cut-story/docs)에 있습니다.
