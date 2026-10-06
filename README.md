# project-tensei
**Reimagining AWS Support: A Shared Workspace for Humans and AI**

Three participants (Customer, AI Agent, Support Engineer), one workspace, continuous collaboration.

---

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Docker | 29+ | [docker.com/get-docker](https://docs.docker.com/get-docker/) |
| Docker Compose plugin | v2+ | [Install Compose](https://docs.docker.com/compose/install/) |
| Git | 2.40+ | `brew install git` or system package manager |
| AWS CLI | v2 | [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| AWS credentials | Configured | `aws configure` or `~/.aws/credentials` |

**Optional (if running without Docker):**
- Python 3.12
- Node.js 20
- AWS CDK (`npm install -g aws-cdk`)
- AWS SAM CLI (`pip install aws-sam-cli`)

---

## Quick Start (Docker — Recommended)

```bash
# 1. Clone the repository
git clone https://github.com/evocasey04/project-tensei.git
cd project-tensei

# 2. Build and start the dev environment
docker compose up -d dev

# 3. Enter the container
docker compose exec dev bash

# 4. (Inside container) Start frontend dev server
cd /app/frontend && npm run dev

# 5. (Inside container, separate terminal) Run backend locally
cd /app/backend && python -m pytest tests/
```

### With LocalStack (mock AWS services locally)

```bash
# Start both dev container and LocalStack
docker compose up -d

# LocalStack endpoints available at http://localhost:4566
# Use --endpoint-url http://localhost:4566 with AWS CLI commands
```

### VS Code Dev Container

1. Install the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
2. Open this repo in VS Code
3. Click "Reopen in Container" when prompted
4. Everything is pre-configured — Python, Node, AWS CLI, extensions

---

## Quick Start (Without Docker)

```bash
# 1. Clone
git clone https://github.com/evocasey04/project-tensei.git
cd project-tensei

# 2. Backend setup
cd backend
python3.12 -m venv .venv
source .venv/bin/activate
pip install -e .[dev]

# 3. Frontend setup
cd ../frontend
npm install
npm run dev

# 4. Infra setup
cd ../infra
npm install
npx cdk synth
```

---

## Project Structure

```
project-tensei/
├── .devcontainer/          # VS Code Dev Container config
├── infra/                  # AWS CDK (TypeScript) — all infrastructure
│   ├── lib/
│   │   ├── network-stack.ts    # VPC, subnets, VPC endpoints
│   │   ├── security-stack.ts   # KMS keys, WAF, IAM policies
│   │   ├── auth-stack.ts       # Cognito user pools, groups
│   │   ├── data-stack.ts       # DynamoDB, S3, EventBridge
│   │   ├── api-stack.ts        # API Gateway HTTP + WebSocket
│   │   ├── compute-stack.ts    # Lambda functions, Step Functions
│   │   └── frontend-stack.ts   # CloudFront + S3 hosting
│   └── bin/app.ts              # CDK app entry point
├── backend/                # Python 3.12 — agent orchestration
│   ├── src/
│   │   ├── agents/
│   │   │   ├── edge/           # Context gathering, skill detection
│   │   │   ├── devops/         # Read-only diagnostics, playbooks
│   │   │   └── aca/            # Scoring, routing, hypothesis generation
│   │   ├── handlers/
│   │   │   ├── api/            # REST endpoint handlers
│   │   │   ├── websocket/      # Real-time connection handlers
│   │   │   └── events/         # EventBridge consumers
│   │   ├── services/
│   │   │   ├── consent.py      # Consent grant/revoke/check
│   │   │   ├── pii.py          # PII detection + redaction
│   │   │   ├── audit.py        # Immutable audit trail
│   │   │   └── ticket.py       # Auto-ticket generation
│   │   ├── models/             # Pydantic schemas (from /schemas)
│   │   └── shared/             # Bedrock client, DynamoDB client, events
│   └── tests/
├── frontend/               # React 18 + TypeScript + Cloudscape
│   ├── src/
│   │   ├── components/         # UI components (see below)
│   │   ├── hooks/              # useWebSocket, useCase, useConsent, useAuth
│   │   ├── store/              # Zustand state management
│   │   └── api/                # Typed API client
│   └── vite.config.ts
├── schemas/                # JSON Schema — single source of truth
│   ├── context-bundle.json
│   ├── case.json
│   ├── event-envelope.json
│   ├── hypothesis.json
│   ├── consent.json
│   ├── diagnostics-report.json
│   └── ticket.json
├── scripts/                # Deploy, type generation, seed data
├── Dockerfile              # Dev environment image
└── docker-compose.yml      # Dev + LocalStack services
```

---

## Git Workflow

**Branch protection is enforced on `main`.**

```bash
# 1. Create a feature branch
git checkout -b feature/your-feature-name

# 2. Do your work, commit
git add <files>
git commit -m "Description of change"

# 3. Push your branch
git push -u origin feature/your-feature-name

# 4. Open a Pull Request on GitHub
gh pr create --title "Your PR title" --body "Description"

# 5. Get 1 approving review, then merge
```

**Rules:**
- No direct pushes to `main` — all changes go through PRs
- 1 approving review required before merge
- Stale reviews are dismissed when new commits are pushed
- All review threads must be resolved before merge
- No force pushes to `main`

---

## Branch Naming Convention

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/<description>` | `feature/edge-agent-prompts` |
| Bug fix | `fix/<description>` | `fix/websocket-reconnect` |
| Infrastructure | `infra/<description>` | `infra/vpc-endpoints` |
| Documentation | `docs/<description>` | `docs/api-contracts` |

---

## Schemas

All data contracts live in `/schemas/` as JSON Schema files. These are the **single source of truth**.

To generate typed models:
```bash
# Generate Pydantic models (backend)
# Generate TypeScript types (frontend)
./scripts/generate-types.sh
```

When modifying a schema, update it in `/schemas/` first, then regenerate.

---

## Environment Variables

Create a `.env` file (git-ignored) for local development:

```env
AWS_DEFAULT_REGION=us-east-1
AWS_PROFILE=your-profile
BEDROCK_MODEL_ID=anthropic.claude-sonnet-4-20250514
LOCALSTACK_ENDPOINT=http://localhost:4566
```

---

## Deploying to AWS

```bash
# From the infra/ directory
cd infra
npm install
npx cdk bootstrap    # First time only
npx cdk deploy --all
```

---

## Security

- All data encrypted at rest (KMS) and in transit (TLS 1.3)
- Customer data isolated by partition key — no cross-tenant access
- Consent required before any resource access, auto-expires after 1 hour
- PII (credentials, keys) auto-detected and redacted before storage
- All access logged to immutable audit trail (S3 Object Lock)
- Lambda functions run in VPC with private subnets
- Bedrock accessed via VPC endpoint (no public internet)

See the development plan for full security architecture details.

---
