# 🚀 GitHub Actions Reusable Workflows

> **Production-ready reusable workflow library for Kubernetes deployments**
> Inspired by real-world CI/CD architecture built for 100+ enterprise applications

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## 🎯 The Problem This Solves

Imagine 100+ microservices, each with its own CI/CD pipeline. Every team writing the same Docker build steps, the same Kubernetes deploy logic, the same approval gates — duplicated across hundreds of YAML files.

**One bug = 100+ files to fix.**
**One security patch = 100+ PRs.**
**One new environment = 100+ changes.**

This library solves that with **one reusable workflow** that all teams call.

---

## 🏗️ Architecture

```
your-app/.github/workflows/cicd.yml  (tenant — just config, no logic)
         │
         └──► github-actions-reusable-workflows (this repo)
                    │
                    ├── reusable-docker-build.yml   ← Build & push image
                    └── reusable-k8s-deploy.yml     ← Deploy to K8s
```

### Flow

```
Code Push
    │
    ▼
Build Docker Image ──► Push to Registry
    │
    ▼
Deploy to E1 (Dev) ──── auto
    │
    ▼
Deploy to E2 (Staging) ─ auto (main branch only)
    │
    ▼
Deploy to E3 (Prod) ──── manual approval required ✋
```

---

## ⚡ Quick Start

### Step 1 — Reference this library in your app

Copy `tenant-template.yml` to your app repo at `.github/workflows/cicd.yml`

```yaml
# Your app only needs configuration — zero pipeline logic
jobs:
  build:
    uses: your-org/github-actions-reusable-workflows/.github/workflows/reusable-docker-build.yml@main
    with:
      image-name: "my-org/my-app"
      image-tag: ${{ github.sha }}
    secrets:
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}

  deploy-prod:
    uses: your-org/github-actions-reusable-workflows/.github/workflows/reusable-k8s-deploy.yml@main
    with:
      environment: "e3"
      image-tag: ${{ github.sha }}
      namespace: "my-app-prod"
      app-name: "my-app"
      require-approval: true    # Manual gate for production
    secrets:
      KUBECONFIG: ${{ secrets.E3_KUBECONFIG }}
```

### Step 2 — Set up secrets in your repo

| Secret | Description |
|--------|-------------|
| `REGISTRY_USERNAME` | Container registry username |
| `REGISTRY_PASSWORD` | Container registry password / token |
| `E1_KUBECONFIG` | Base64 encoded kubeconfig for Dev cluster |
| `E2_KUBECONFIG` | Base64 encoded kubeconfig for Staging cluster |
| `E3_KUBECONFIG` | Base64 encoded kubeconfig for Production cluster |

Encode your kubeconfig:
```bash
cat ~/.kube/config | base64 -w 0
```

### Step 3 — Push and watch it run 🎉

---

## 📦 Available Workflows

### `reusable-docker-build.yml`

Builds and pushes Docker images with caching, metadata and multi-tag support.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image-name` | ✅ | — | Docker image name |
| `registry` | ❌ | `ghcr.io` | Container registry |
| `image-tag` | ❌ | `${{ github.sha }}` | Image tag |
| `dockerfile-path` | ❌ | `./Dockerfile` | Path to Dockerfile |
| `push` | ❌ | `true` | Push to registry |

**Outputs:** `image-tag`, `image-digest`

---

### `reusable-k8s-deploy.yml`

Deploys to Kubernetes using Helm with environment gates and rollback support.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `environment` | ✅ | — | Target environment (e1/e2/e3) |
| `image-tag` | ✅ | — | Docker image tag |
| `namespace` | ✅ | — | Kubernetes namespace |
| `app-name` | ✅ | — | Application name |
| `replicas` | ❌ | `2` | Number of replicas |
| `require-approval` | ❌ | `false` | Manual approval gate |
| `helm-chart-path` | ❌ | `./helm` | Helm chart path |

---

## 🔒 Production Safety Features

- **Manual approval gate** for E3 (production) — no accidental prod deployments
- **`--atomic` Helm flag** — auto rollback if deployment fails
- **Deployment verification** — waits for rollout to complete before success
- **GitHub build summaries** — rich deployment info in every run
- **Docker layer caching** — faster builds on repeat runs

---

## 📊 Real World Impact

This pattern was used to standardise CI/CD for **100+ enterprise microservices**:

| Before | After |
|--------|-------|
| 40 min average deployment | 5 min average deployment |
| Duplicate pipeline logic in every repo | Single source of truth |
| Manual deployments to production | Automated with approval gate |
| No standardisation across teams | Consistent across all 100+ apps |

---

## 🗂️ Repository Structure

```
.
└── .github/
    └── workflows/
        ├── reusable-docker-build.yml   # Docker build & push
        ├── reusable-k8s-deploy.yml     # Kubernetes deployment
        └── tenant-template.yml         # Copy this to your app repo
```

---

## 🤝 Contributing

1. Fork this repo
2. Create a feature branch (`git checkout -b feature/add-sonar-scan`)
3. Commit changes (`git commit -m 'Add SonarQube scan step'`)
4. Push and open a PR

---

## 📄 License

MIT License — free to use, modify and distribute.

---

**Built by [Selvathalapathy](https://linkedin.com/in/selvathalapathy) — Senior DevOps / Platform Engineer**
*10+ years experience | CKA | AZ-400 | GCP (in progress)*
