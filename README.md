# common

**서비스들이 함께 쓰는 계약과 빌드 설정의 원본이에요. OpenAPI 계약, parent POM, 재사용 CI.**

[← 4cut story 소개](https://github.com/4cut-story) · [아키텍처](https://github.com/4cut-story/.github/blob/main/docs/architecture.md)

## 하는 일

| 경로 | 내용 |
| --- | --- |
| `contracts/` | 서비스별 OpenAPI, 공용 스키마, 게이트웨이 라우트 표 |
| `parent-pom/` | Spring 서비스 네 개가 쓰는 parent POM (GitHub Packages로 배포) |
| `quality/` | 정적 분석 규칙 (Spotless, SpotBugs, PMD) |
| `.github/workflows/` | Java·Node·Python 서비스용 재사용 CI 워크플로 |
| `templates/` | 서비스 CI와 Dockerfile 템플릿 |
| `service-catalog.yaml` | 서비스 이름, 포트, 주소, 패키지 이름의 원본 |

각 서비스는 이 저장소의 **태그된 버전**에 고정해서 써요.

## 시스템 안에서의 위치

```
common ──계약·parent POM·CI──> web, api-gateway, auth-service, conversation-service, content-service, model-runtime
                                 └─ main 머지 → GHCR 이미지 → deploy 태그 갱신 → Argo CD 배포
```

## 계약을 바꾸는 순서

`contracts/` PR → 태그 → 제공하는 서비스 구현 → 부르는 서비스 업데이트. 하위 호환이 깨지면 `/v2` 경로로 올려요.

## CI와 품질 기준

모든 서비스는 빌드 도구만으로 품질을 검사하고, 하나라도 어기면 CI가 실패해요. **테스트 커버리지는 모든 서비스 80% 이상**이에요.

| 대상 | 재사용 워크플로 | 검사 |
| --- | --- | --- |
| Spring 서비스 4개 | `java-service.yml` | 테스트, Spotless, SpotBugs + FindSecBugs, PMD, CPD, JaCoCo 80% |
| web | `node-service.yml` | ESLint, tsc, Prettier, jscpd, Vitest 80%, `next build` |
| model-runtime | `python-service.yml` | ruff, mypy, pytest-cov 80% |

parent POM은 `parent-pom/`·`quality/` 변경이 main에 머지되면 `release.yml`이 태그를 만들고 배포해요.

## 자세한 문서

| 문서 | 내용 |
| --- | --- |
| [빌드 설정과 CI 기준](docs/build-and-ci.md) | Spring 서비스 시작 규칙, pom.xml 예시, API 문서(REST Docs), CI 단계와 배포 연결 |
| [계약](contracts/README.md) | OpenAPI 계약 목록 |
| [아키텍처](https://github.com/4cut-story/.github/blob/main/docs/architecture.md) | 서비스별 역할과 통신 방식, 배포 흐름 |
