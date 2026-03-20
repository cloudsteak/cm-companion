# CM-Companion

This repository contains the configuration and deployment manifests for the Cloud Mentor platform services, managed by ArgoCD.

## Overview

The `cm-companion` project uses an "App of Apps" pattern to manage infrastructure and application components. Currently, it focuses on the **production** environment. It integrates with AWS Secrets Manager and Parameter Store using the External Secrets Operator (ESO) for secure secret management.

## Applications

### Evolvia Backend (Production)
The core backend infrastructure for the Evolvia platform.
- **`backend-core`**: The main API service.
- **`backend-cleanup-trigger`**: A scheduled CronJob for maintenance and cleanup tasks.
- **`evolvia-verify-lab`**: Service for verifying laboratory data.
- **`redis`**: In-cluster Redis instance for caching and session management.
- **`shared-secrets`**: Management of platform-wide credentials (e.g., GHCR auth).
- **`namespace`**: Defines the `evolvia-backend-prod` and `evolvia-backend-cronjobs-prod` namespaces.

### CM-Messenger (Production)
A dedicated service for handling platform communications and notifications.
- **`messenger`**: The core messaging service.
- **`secrets`**: ExternalSecrets for fetching API keys and configuration.
- **`namespace`**: Defines the `cm-messenger-prod` namespace.

### Cluster Autoscaler (Production)
Manages the automatic scaling of Kubernetes nodes.
- **`civo-config`**: Configuration specific to the Civo cloud provider for cluster autoscaling.

## Repository Structure

```
.
├── apps/
│   └── production/             <-- Active production environment
│       ├── cluster-autoscaler/
│       ├── cm-messenger/
│       └── evolvia-backend/    <-- Main application suite
```

## Deployment Pattern

Each application component typically follows this structure:
- `application.yaml`: ArgoCD Application manifest for the component.
- `values.yaml`: Helm values (if the component uses a Helm chart).
- `secrets/`: Sub-application for managing ExternalSecrets.
- `manifests/`: Raw Kubernetes manifests (e.g., VirtualServices, Gateways, ExternalSecrets).

Components use **Sync Waves** to ensure a correct deployment order:
1.  **Wave -1**: Namespaces.
2.  **Wave 0**: Secrets and configuration.
3.  **Wave 1**: Main application services (Deployments, CronJobs).
4.  **Wave 2**: Ingress and networking components (e.g., `evolvia-backend-core-ingress`, VirtualServices, Gateways).
