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

## CI와 품질 기준

모든 서비스는 SonarQube 없이 빌드 도구만으로 품질을 검사하고, 하나라도 어기면 CI가 실패한다. **테스트 커버리지는 모든 서비스 80% 이상.**

| 대상 | 재사용 워크플로 | 검사 |
|---|---|---|
| Spring 서비스 4개 | `.github/workflows/java-service.yml` | parent POM의 `mvn verify`: 테스트, Spotless(google-java-format), SpotBugs + FindSecBugs, PMD, CPD(중복), JaCoCo 라인 80%, maven-enforcer |
| web | `.github/workflows/node-service.yml` | ESLint(경고도 실패), `tsc --noEmit`, Prettier, jscpd(중복), Vitest 커버리지 80%, `next build` |
| model-runtime | `.github/workflows/python-service.yml` | ruff check, ruff format, mypy, pytest-cov 80% |

- PR과 main push 모두 검사한다. main push일 때만 GHCR에 `:<sha>` 이미지를 올린다. `deploy` 레포의 `sync-images.yml`이 10분마다(또는 수동 실행) 각 서비스의 마지막 성공 이미지를 찾아 `k8s/apps/<서비스>/kustomization.yaml` 태그를 자동 커밋하고, Argo CD가 자동 동기화한다. 코드(pom.xml·package.json·pyproject.toml)가 없는 레포는 `detect` 잡이 검사를 건너뛴다. (model-runtime은 GPU PC compose라 이미지만 만든다.)
- 정적 분석 규칙: `quality/` (parent POM이 같은 버전 태그의 파일을 읽는다)
- 템플릿: `templates/ci.yml`, `templates/Dockerfile.java`
- parent POM 배포: `parent-pom/`·`quality/` 변경이 main에 머지되면 `release.yml`이 pom 버전으로 태그 `vX.Y.Z`를 만들고 GitHub Packages에 배포한다. 같은 버전이 이미 있으면 건너뛴다.
- 필요한 시크릿: 없음. 모든 워크플로가 기본 `GITHUB_TOKEN`만 쓴다. 공개 레포의 패키지는 공개라서 서비스 레포가 자기 `GITHUB_TOKEN`으로 parent POM을 읽는다.

### crowfoot-auth CI에서 바꾼 점
- **테스트를 CI에서 돌린다** (`-DskipTests` 제거): 테스트는 머지 조건이다 (constitution V).
- **PR에서도 돈다**: 머지 전에 실패를 잡는다. 이미지와 배포는 main push일 때만.
- **배포 태그는 deploy 레포가 가져간다(pull)**: crowfoot는 서비스 CI가 매니페스트 레포에 직접 커밋해 레포 간 쓰기 토큰(PAT)이 필요했다. 여기서는 deploy 레포가 공개 API로 성공한 빌드를 찾아 자기 `GITHUB_TOKEN`으로 커밋한다. 개인 토큰 만료·유출 걱정이 없고, 대신 반영까지 최대 10분 정도 걸린다.
- **`latest` 태그를 쓰지 않는다**: 떠 있는 이미지가 어느 커밋인지 sha로만 식별한다. 롤백은 deploy 레포 커밋 되돌리기.
- **재사용 워크플로 한 곳에서 관리**: 서비스마다 같은 단계를 복사하지 않는다.
- **품질 검사를 parent POM에**: CI와 로컬 `mvn verify`가 같은 기준으로 돈다. 커버리지 80%, 스타일 검사를 더했다.
- **`sed` 대신 `kustomize edit set image`**: deployment.yaml 형식에 기대지 않는다.
- **jar를 한 번만 빌드**: verify에서 만든 jar를 이미지에 그대로 넣는다.
- **규칙 파일을 버전 태그로 고정**: 규칙을 바꿔도 이전 parent 버전을 쓰는 서비스 빌드가 깨지지 않는다.
