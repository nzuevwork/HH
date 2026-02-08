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

## Reliability goals

Инфраструктура спроектирована так, чтобы минимизировать простой сервиса.

Реализовано:
- Rolling update без downtime
- Автоматическое восстановление pod'ов Kubernetes
- Readiness и liveness probes
- Мониторинг доступности сервиса через Prometheus
- Быстрое обнаружение сбоев через Grafana dashboards

Цель: при ошибочном деплое сервис автоматически восстанавливается или может быть быстро откатан.

---
## Incident response example

Ситуация:
После обновления приложения веб-сервис стал недоступен.

Действия:

1. Проверил доступность ingress (Traefik) — маршрут существует
2. Проверил service — endpoint присутствует
3. kubectl get pods — pod в состоянии CrashLoopBackOff
4. kubectl describe pod — контейнер не проходит readiness probe
5. kubectl logs — ошибка конфигурации приложения

Причина:
Некорректное значение переменной окружения в Kubernetes Secret.

Решение:
Исправлен Secret и выполнено:
kubectl rollout restart deployment

Результат:
Pod успешно перезапущен, сервис восстановлен.
---
## Operational experience

Кластер используется постоянно. 
При каждом обновлении приложения выполняется rolling update без остановки сервиса.
Мониторинг Grafana используется для отслеживания состояния pod’ов и доступности веб-сервиса.

В случае ошибки деплоя выполняю диагностику через:
kubectl describe, kubectl logs, проверку ingress и сервисов, после чего выполняется исправление конфигурации или rollback.

---
Секреты (токен реестра, kubeconfig) хранятся в секретах GitHub Actions.
Они внедряются в задания CI во время выполнения и не хранятся в репозитории.
Самостоятельно размещенный раннер предназначен исключительно для CI/CD и не имеет публичного доступа
