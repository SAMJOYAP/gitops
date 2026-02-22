# SAMJOYAP GitOps Repository

This repository stores Kubernetes manifests for application deployment.

## Layout

- `apps/node-express/manifests/deployment.yaml`
- `apps/java-spring/manifests/deployment.yaml`
- `apps/backstage-kr/manifests/deployment.yaml`
- `apps/*/manifests/{namespace,service,ingress}.yaml`

## CD update targets

Application repositories should update only these files:

- `apps/node-express/manifests/deployment.yaml`
- `apps/java-spring/manifests/deployment.yaml`
- `apps/backstage-kr/manifests/deployment.yaml`

The CD workflow should replace the `image:` field in each deployment and open a PR to this repository.

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
