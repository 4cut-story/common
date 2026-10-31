# contracts

서비스 간 계약의 원본입니다. 각 서비스는 이 레포의 **태그된 버전**에 고정해 쓰고, CI는 그 버전으로 테스트합니다.

| 파일 | 내용 | 작업 |
|---|---|---|
| `conversation-service.yaml` | 생성 흐름 외부·내부 API | 001 T011 |
| `model-runtime.yaml` | `/transcribe`, `/images` | 001 T011 |
| `auth-service.yaml` | 로그인, 회원, JWKS | 002 T010 |
| `content-service.yaml` | 만화함, 게시, 피드 | 002 T011 |
| `schemas/pipeline.yaml` | Transcript, SceneCandidate, ComicScript | 001 T012 |
| `schemas/common.yaml` | 공통 오류 형식, 커서 페이지, 공통 헤더·파라미터·오류 응답 | 002 T012 ✅ |
| `gateway-routes.md` | 게이트웨이 라우트 표 | 002 T013 |

모든 계약은 `docs` 레포의 [API 공통 규칙](https://github.com/4cut-story/docs/blob/main/docs/api-conventions.md)을 따르고, 오류·페이지·헤더는 `schemas/common.yaml`을 `$ref`로 가져다 쓴다. 공용 DTO는 손으로 쓰지 않고 이 계약에서 생성한다.

첫 계약 묶음이 준비되면 `v0.1.0`으로 태그합니다.
