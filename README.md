# DevOps Homelab — Production-like CI/CD Kubernetes Infrastructure

## Overview
Проект представляет собой production-подобную инфраструктуру развертывания веб-сервисов.

Pipeline автоматически доставляет приложение от git push до публично доступного веб-сервиса с мониторингом.

GitHub → CI → Docker → Registry → Kubernetes → Ingress → Public service → Monitoring

---

## Architecture

1. Разработчик делает push в GitHub
2. GitHub Actions (self-hosted runner) запускает pipeline
3. Собирается Docker image
4. Образ публикуется в GitHub Container Registry (GHCR)
5. Kubernetes обновляет Deployment
6. Traefik публикует сервис наружу
7. Prometheus собирает метрики
8. Grafana отображает состояние сервиса

---

## Infrastructure

- Kubernetes: k3s cluster (control-plane + worker)
- CI/CD: GitHub Actions self-hosted runner
- Registry: GHCR
- Ingress: Traefik
- Monitoring: Prometheus + node-exporter + kube-state-metrics + Grafana

---

