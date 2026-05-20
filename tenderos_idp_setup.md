# TenderOS IDP — Platform Engineering Layer Setup

## What Was Built

The TenderOS Internal Developer Platform (IDP) has been fully scaffolded at:
`d:\TenderOS\platform-engineering\`

---

## Project Structure

```
platform-engineering/
├── app-config.yaml                    ✅ Full TenderOS config with all integrations
├── app-config.production.yaml         ✅ Production overrides (PostgreSQL, GCS TechDocs)
├── .env.example                       ✅ All env vars documented with instructions
├── README.md                          ✅ Comprehensive setup guide
│
├── catalog/
│   ├── org.yaml                       ✅ 5 teams + platform admin user
│   ├── systems.yaml                   ✅ 3 domains, 5 systems, 9 service components
│   └── apis.yaml                      ✅ GraphQL API schema + OpenAPI spec
│
├── templates/                         ✅ 5 Golden Path Templates
│   ├── nestjs-microservice/           ✅ NestJS + Kafka + K8s + CI/CD
│   ├── python-worker/                 ✅ FastAPI/Celery/Kafka + AI option
│   ├── kafka-consumer/               ✅ Dedicated consumer with DLQ
│   ├── nextjs-app/                   ✅ Portal/Dashboard/Auction interface
│   └── internal-tool/                 ✅ CLI/batch/migration tools
│
├── kubernetes/
│   └── backstage-rbac.yaml            ✅ ClusterRole + ServiceAccount for K8s plugin
│
└── packages/
    ├── app/src/modules/nav/
    │   ├── LogoFull.tsx               ✅ TenderOS branded logo (shield + wordmark)
    │   └── LogoIcon.tsx               ✅ TenderOS icon (collapsed sidebar)
    └── backend/src/index.ts           ✅ All backend plugins registered
```

---

## Integration Configuration Map

All secrets go in `.env` (copy from `.env.example`):

| Integration | Env Vars Needed | Catalog Annotation |
|---|---|---|
| **GitHub** | `GITHUB_TOKEN`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` | `github.com/project-slug` |
| **Kubernetes** | `K8S_PROD_CLUSTER_URL`, `K8S_PROD_SERVICE_ACCOUNT_TOKEN` | `backstage.io/kubernetes-id` |
| **ArgoCD** | `ARGOCD_BASE_URL`, `ARGOCD_AUTH_TOKEN` | `argocd/app-name` |
| **Jenkins** | `JENKINS_BASE_URL`, `JENKINS_BASIC_AUTH` | `jenkins.io/job-full-name` |
| **SonarQube** | `SONARQUBE_BASE_URL`, `SONARQUBE_TOKEN` | `sonarqube.org/project-key` |
| **PagerDuty** | `PAGERDUTY_TOKEN` | `pagerduty.com/service-id` |
| **Kafka UI** | `KAFKA_UI_BASE_URL` | (via proxy) |

---

## Kafka Event Taxonomy

The Kafka consumer template includes the full TenderOS event schema:

| Topic | Publisher | Description |
|---|---|---|
| `tender.discovered` | Tender Intelligence Engine | New tender found |
| `bid.created` | Workflow Engine | New bid initiated |
| `bid.status.changed` | Workflow Engine | Bid status transition |
| `document.uploaded` | Document Engine | Document processed |
| `auction.started` | Workflow Engine | Reverse auction began |
| `auction.bid.placed` | Client Portal | Bid placed in auction |
| `workflow.transition` | Workflow Engine | Workflow state change |
| `notification.triggered` | Any service | Notification event |
| `compliance.event` | Analytics Service | Audit trail event |

---

## TenderOS Catalog Entities

### Domains → Systems → Components

```
tender-lifecycle domain
  └── tenderos-core system
        ├── api-gateway (NestJS GraphQL BFF)
        ├── iam-service (Keycloak-backed IAM)
        ├── workflow-engine (bid lifecycle)
        ├── event-processor (Kafka backbone)
        ├── notification-engine (Firebase + SendGrid)
        └── analytics-service (BI + audit)
  └── tender-intelligence system
        └── tender-intel-engine (Python FastAPI + AI)
        
client-interaction domain
  └── document-exchange system
        └── document-engine (PDF, signed URLs)
  └── client-portal system
        └── client-portal-app (Next.js)

platform-engineering domain
  └── platform-idp system
        └── (this Backstage instance)
```

---

## Installation Status

> [!IMPORTANT]
> `yarn install` is currently running in the background.
> It compiles native modules (`better-sqlite3`, `node-gyp`) which takes **10–30 minutes** on Windows.
> **Recommended**: Run this in WSL2 for significantly faster builds.

### Once installation completes:

```bash
# 1. Set up environment
cp .env.example .env
# Edit .env with your actual keys

# 2. Start the IDP
yarn start
# → Frontend: http://localhost:3000
# → Backend:  http://localhost:7007
```

### Install Integration Plugins (Phase 2)

After the base install works, add these frontend plugins:

```bash
# ArgoCD GitOps visibility
yarn workspace app add @roadiehq/backstage-plugin-argo-cd

# SonarQube code quality cards
yarn workspace app add @backstage-community/plugin-sonarqube
yarn workspace backend add @backstage-community/plugin-sonarqube-backend

# PagerDuty incident management
yarn workspace app add @pagerduty/backstage-plugin
yarn workspace backend add @pagerduty/backstage-plugin-backend

# Kafka topic visibility
yarn workspace app add @backstage-community/plugin-kafka
yarn workspace backend add @backstage-community/plugin-kafka-backend
```

---

## Key URLs (once running)

| Page | URL |
|---|---|
| Home / Catalog | http://localhost:3000 |
| Software Catalog | http://localhost:3000/catalog |
| Golden Path Templates | http://localhost:3000/create |
| API Docs | http://localhost:3000/api-docs |
| TechDocs | http://localhost:3000/docs |
| System Graph | http://localhost:3000/catalog-graph |
| Search | http://localhost:3000/search |
| Settings | http://localhost:3000/settings |

---

## Next Steps

1. **Complete `yarn install`** (running in background — check terminal)
2. **Set up `.env`** with your actual tokens
3. **Apply K8s RBAC**: `kubectl apply -f kubernetes/backstage-rbac.yaml`
4. **Start**: `yarn start`
5. **Install integration plugins** (Phase 2, listed above)
6. **Register GitHub repos** via Catalog Import or add `catalog-info.yaml` to each service repo
7. **Customize golden path skeletons** with your actual boilerplate code

> [!TIP]
> For production: Use WSL2 or a Linux CI runner for `yarn install`. The Windows native module compilation is notoriously slow. Once built, Docker images work fine on any platform.
