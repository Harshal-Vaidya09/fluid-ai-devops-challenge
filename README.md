# Fluid AI DevOps Challenge

Production-style Kubernetes and CI/CD demonstration.

## Stack

- Java 17
- Spring Boot 3.5.16
- Maven
- Docker
- Kubernetes
- Minikube
- PostgreSQL 16
- Jenkins
- GitHub

## Architecture

Spring Boot backend deployed to Kubernetes with PostgreSQL as the database dependency.

The deployment includes:

- Kubernetes Deployments
- ClusterIP Services
- PersistentVolumeClaim
- Kubernetes Secrets
- Liveness probes
- Readiness probes
- Jenkins CI/CD

## Application

Backend API:

`/api/hello`

Health endpoints:

`/actuator/health`

`/actuator/health/liveness`

`/actuator/health/readiness`