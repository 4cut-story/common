# common

parent POM, 서비스 간 계약(OpenAPI), 서비스 카탈로그의 원본 레포다.

- 상위 원칙: `docs` 레포의 `.specify/memory/constitution.md`
- 설계: `docs` 레포의 `docs/architecture.md` (v3.0) 4·5·13절
- 계약은 태그로 배포한다. 첫 계약 묶음은 `v0.1.0`. 하위 호환이 깨지면 `/v2`.
- Java groupId는 `net.java21.fourcut`, 패키지는 `net.java21.fourcut.<서비스>`. parent POM `0.2.0`부터 적용 (`0.1.0`은 옛 groupId `com.fourcut`).
- parent POM 버전은 Boot 4 호환표(`docs` 레포 `docs/versions.md`)를 확인한 뒤 고정한다.
- 서비스 이름·포트를 바꾸면 `deploy` 레포의 compose·매니페스트도 함께 고친다.
