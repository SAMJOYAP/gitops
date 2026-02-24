# SAMJOYAP GitOps 저장소

이 저장소는 애플리케이션의 GitOps 배포 상태를 관리합니다.

## 디렉터리 구조

- `apps/<appname>/`
  - `kustomization.yaml`
  - `manifests/`
    - `namespace.yaml`
    - `service.yaml`
    - `deployment.yaml`
    - `ingress.yaml` (옵션)
- `apps/backstage-already11/` (Backstage 전용 values 기반)
  - `values.yaml`
  - `values-already11.yaml`
  - `kustomization.yaml`
  - `manifests/`
    - `external-secrets.yaml`
    - `k8s-config-secret.yaml`

## CD 업데이트 대상

애플리케이션 저장소 CD는 아래 파일만 업데이트해야 합니다.

- `apps/<appname>/manifests/deployment.yaml`
- `apps/backstage-already11/values-already11.yaml`

- 일반 앱 CD는 Deployment의 `image:`를 교체합니다.
- Backstage CD는 `backstage.image.registry/repository/tag`를 교체합니다.
- 모든 변경은 PR 기반으로 반영됩니다(auto-merge).

## 자동 머지 전제 조건

GitOps PR 자동 머지를 사용하려면 아래 조건이 필요합니다.

- GitOps repo `Settings -> General -> Pull Requests`
  - `Allow auto-merge` 활성화
- GitOps repo `Settings -> Actions -> General`
  - `Workflow permissions: Read and write permissions`
  - `Allow GitHub Actions to create and approve pull requests` 활성화
- `GITOPS_REPO_TOKEN` 권한
  - `Contents: Read and write`
  - `Pull requests: Read and write`
- Ruleset/Branch protection 사용 시
  - CD 토큰 주체(사용자 또는 GitHub App) bypass 허용

## Argo CD 소스

Argo CD의 소스로 이 저장소를 사용합니다.

권장 경로:

- 단일 앱 배포:
  - `apps/<appname>`
  - `apps/backstage-already11`
- 전체 앱 배포:
  - `apps`

## 호스트

- `https://<appname>.sesac.already11.cloud`
- `https://bs.sesac.already11.cloud`

## 최신 상태 메모 (2026-02-23)

### 1) 앱 운영 기준

- `backstage-kr` 대신 `backstage-already11`을 운영 기준 앱으로 사용한다.
- GitOps 경로는 `apps/backstage-already11`로 고정해 관리한다.

### 2) 템플릿 반영 경로

- Backstage 템플릿(Node.js, Java) 한글화 및 EKS 선택 기능은
  `reference-implementation-aws/templates/backstage` 소스 기준으로 관리한다.
- 운영 UI에 반영되려면 Backstage 앱의 catalog source가 최신 템플릿 저장소를 가리켜야 한다.

### 3) 반영 확인 포인트

1. `apps/backstage-already11/values-already11.yaml` 이미지 태그 최신 여부
2. Backstage 환경변수(`APP_CONFIG_*`)로 catalog location override 적용 여부
3. Argo CD sync 완료 후 Template 화면에서 한글 문구/EKS Cluster 필드 노출 여부

### 4) BACKSTAGE_API_TOKEN 연동 (2026-02-23)

템플릿 자동 반영 워크플로우에서 Backstage Catalog API 인증을 위해 아래를 반영함:

- `apps/backstage-already11/values.yaml`
  - `backstage.appConfig.backend.auth.externalAccess` 추가
  - 토큰 참조: `${BACKSTAGE_API_TOKEN}`
- `apps/backstage-already11/manifests/external-secrets.yaml`
  - `backstage-env-vars`에 `BACKSTAGE_API_TOKEN` 주입 매핑 추가
  - source: `keycloak` secret store의 `keycloak-clients.BACKSTAGE_API_TOKEN`

운영 주의:
- `backstage-env-vars`는 DB 비밀번호(`POSTGRES_PASSWORD`)도 포함하므로
  Secret 재생성/수동 패치 시 DB 인증 불일치가 나지 않도록 주의해야 함.

### 5) Argo CD RBAC 운영 경계 (중요)

- 이 저장소는 애플리케이션 배포 상태를 관리하며,
  Argo CD 컨트롤 플레인 자체(`argocd-rbac-cm`)는 직접 소유하지 않는다.
- 따라서 Backstage에서 Argo API를 안정적으로 사용하려면
  Argo CD 운영 경로(Argo 설치/운영 레포 또는 클러스터 직접 적용)에서
  아래 RBAC 정책을 별도로 반영해야 한다.

예시 (`argocd-rbac-cm`의 `policy.csv`):

```csv
p, role:backstage, applications, get, */*, allow
p, role:backstage, applications, list, */*, allow
p, role:backstage, projects, get, *, allow
p, role:backstage, projects, list, *, allow
p, role:backstage, clusters, get, *, allow
p, role:backstage, clusters, list, *, allow
p, role:backstage, clusters, create, *, allow
g, backstage, role:backstage
```

- 그리고 Backstage가 사용하는 Argo 토큰(`ARGOCD_AUTH_TOKEN`)이
  위 role이 매핑된 계정 토큰과 일치해야 한다.

### 6) Backstage CSP 운영 설정 (2026-02-24)

Backstage 운영값은 `apps/backstage-already11/values.yaml`에서 관리한다.

- 반영 항목:
  - `backstage.appConfig.backend.csp.connect-src`: `["'self'", "http:", "https:"]`
  - `backstage.appConfig.backend.csp.img-src`: `["'self'", "data:", "https://cdn.simpleicons.org", "https://a0.awsstatic.com"]`
  - `self` 토큰은 반드시 `"'self'"` 형태로 지정해야 same-origin 이미지(예: favicon)가 차단되지 않음

운영 주의:
- CSP 변경은 Backstage 재배포(Argo CD sync) 후 적용된다.
- 값 미반영 시 브라우저 콘솔에 `Content Security Policy` 이미지 차단 로그가 발생할 수 있다.
