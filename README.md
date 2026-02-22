# SAMJOYAP GitOps Repository

This repository stores GitOps deployment state for apps.

## Layout

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

## CD update targets

Application repositories should update only these files:

- `apps/node-express/manifests/deployment.yaml`
- `apps/java-spring/manifests/deployment.yaml`
- `apps/backstage-kr/values-kr.yaml`

- Node/Java CD replaces Deployment `image:`.
- Backstage CD replaces `backstage.image.registry/repository/tag`.
- All updates are applied via PR (auto-merge).

## Argo CD source

Use this repository as Argo CD source.

Recommended paths:

- Single app deployment:
  - `apps/node-express`
  - `apps/java-spring`
  - `apps/backstage-kr`
- Deploy both apps:
  - `apps`

## Hosts

- `https://node-express.sesac.already11.cloud`
- `https://java-spring.sesac.already11.cloud`
- `https://backstage-kr.sesac.already11.cloud`
