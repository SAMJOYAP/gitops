# SAMJOYAP GitOps 저장소

이 저장소는 애플리케이션의 GitOps 배포 상태를 관리합니다.

## 디렉터리 구조

- `apps/node-express/`
  - `kustomization.yaml`
  - `manifests/{namespace,service,deployment,ingress}.yaml`
- `apps/java-spring/`
  - `kustomization.yaml`
  - `manifests/{namespace,service,deployment,ingress}.yaml`
- `apps/backstage-kr/`
  - `values.yaml`
  - `values-kr.yaml`
  - `kustomization.yaml`
  - `manifests/{external-secrets,k8s-config-secret}.yaml`

## CD 업데이트 대상

애플리케이션 저장소 CD는 아래 파일만 업데이트해야 합니다.

- `apps/node-express/manifests/deployment.yaml`
- `apps/java-spring/manifests/deployment.yaml`
- `apps/backstage-kr/values-kr.yaml`

- Node/Java CD는 Deployment의 `image:`를 교체합니다.
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
  - `apps/node-express`
  - `apps/java-spring`
  - `apps/backstage-kr`
- 전체 앱 배포:
  - `apps`

## 호스트

- `https://node-express.sesac.already11.cloud`
- `https://java-spring.sesac.already11.cloud`
- `https://backstage-kr.sesac.already11.cloud`
