# Docker Setup - ASX Trading AI + 99.ai CRM

## Overview

This Docker setup runs both systems in containers:
1. **ASX Trading AI** - Automated trading system with DynamoDB backend
2. **99.ai CRM API** - Client relationship management FastAPI server

## Quick Start

### 1. Prerequisites

- Docker installed ([Download Docker Desktop](https://www.docker.com/products/docker-desktop))
- AWS credentials configured (already done - in `.env` file)

### 2. Build Images

```bash
cd C:\Users\riord
docker-compose build
```

This builds both images:
- `asx-trading-ai` (Python trading bot)
- `99ai-crm-api` (FastAPI server)

### 3. Run Containers

```bash
# Start both services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f asx-trading
docker-compose logs -f crm-api
```

### 4. Access Services

- **CRM API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs (interactive Swagger UI)
- **Health Check**: http://localhost:8000/health

### 5. Stop Services

```bash
# Stop containers
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

## Architecture

```
┌─────────────────────────────────────┐
│       Docker Compose Network        │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────────────────────────┐  │
│  │   ASX Trading AI Container   │  │
│  │  - Python trading bot        │  │
│  │  - Runs continuously         │  │
│  │  - DynamoDB integration      │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   99.ai CRM API Container    │  │
│  │  - FastAPI server            │  │
│  │  - Port 8000 exposed         │  │
│  │  - DynamoDB integration      │  │
│  └──────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
           ↓
    AWS DynamoDB (7 tables)
```

## Environment Variables

All AWS credentials are in `.env` file:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_DEFAULT_REGION`

**Security**: Never commit `.env` to git (already in .gitignore)

## Individual Services

### ASX Trading AI

**Build only:**
```bash
cd asx-trading-ai
docker build -t asx-trading .
```

**Run standalone:**
```bash
docker run -d \
  --name asx-trading \
  --env-file ../.env \
  asx-trading
```

### 99.ai CRM API

**Build only:**
```bash
cd projects/ai-consulting-business
docker build -t 99ai-crm .
```

**Run standalone:**
```bash
docker run -d \
  --name crm-api \
  -p 8000:8000 \
  --env-file ../../.env \
  99ai-crm
```

## Debugging

### View logs

```bash
# Follow logs in real-time
docker-compose logs -f

# Last 100 lines
docker-compose logs --tail=100

# Specific service
docker-compose logs -f asx-trading
docker-compose logs -f crm-api
```

### Exec into container

```bash
# Open shell in trading container
docker-compose exec asx-trading /bin/bash

# Open shell in CRM container
docker-compose exec crm-api /bin/bash
```

### Check health

```bash
# Container status
docker-compose ps

# CRM API health endpoint
curl http://localhost:8000/health

# View resource usage
docker stats
```

## Deployment to EC2

### Option 1: Docker Compose (Simple)

1. Copy these files to EC2:
   - `docker-compose.yml`
   - `.env`
   - `asx-trading-ai/` (entire directory)
   - `projects/ai-consulting-business/` (entire directory)

2. Run on EC2:
   ```bash
   docker-compose up -d
   ```

### Option 2: Docker Hub (Recommended)

1. Push images to Docker Hub:
   ```bash
   docker tag asx-trading riord/asx-trading:latest
   docker tag 99ai-crm riord/99ai-crm:latest

   docker push riord/asx-trading:latest
   docker push riord/99ai-crm:latest
   ```

2. Pull and run on EC2:
   ```bash
   docker pull riord/asx-trading:latest
   docker pull riord/99ai-crm:latest
   docker-compose up -d
   ```

### Option 3: AWS ECR (Enterprise)

1. Create ECR repositories
2. Push images to ECR
3. Use ECS/Fargate for orchestration

## Files Created

```
C:\Users\riord\
├── docker-compose.yml        # Orchestrates both services
├── .env                       # AWS credentials (not in git)
├── .env.example              # Template for credentials
│
├── asx-trading-ai/
│   ├── Dockerfile            # ASX Trading image
│   ├── .dockerignore         # Exclude unnecessary files
│   ├── requirements.txt      # Python dependencies (boto3 added)
│   └── ... (all Python files)
│
└── projects/ai-consulting-business/
    ├── Dockerfile            # CRM API image
    ├── .dockerignore         # Exclude unnecessary files
    ├── requirements.txt      # FastAPI + boto3
    ├── crm_api.py           # FastAPI server (NEW)
    └── ai_crm_dynamodb.py   # CRM helper (renamed from 99ai_*)
```

## Next Steps

1. ✅ Build images locally → ✅ COMPLETE (2025-10-28)
2. ✅ Test locally → ✅ COMPLETE (Both services operational)
3. ⏭️ Deploy to EC2 → Run 24/7 in cloud
4. ⏭️ Set up Route53 → Custom domain (99.ai)
5. ⏭️ Add SSL/HTTPS → Secure connections

## Local Deployment Results (2025-10-28)

**Build Success:**
- `riord-asx-trading`: 819MB (Python 3.11 + trading system)
- `riord-crm-api`: 580MB (Python 3.11 + FastAPI)
- Build time: ~2 minutes

**Runtime Status:**
- ASX Trading AI: ✅ Running, executing live trades
- 99.ai CRM API: ✅ Running on http://localhost:8000
- Health checks: ✅ Both services healthy
- DynamoDB: ✅ Connected and operational

**Resource Usage:**
- ASX Trading: 69MB RAM (0.89% of 8GB)
- CRM API: 59MB RAM (0.76% of 8GB)
- Total: ~128MB RAM (very efficient)

**Verified Functionality:**
- ✅ API docs accessible: http://localhost:8000/docs
- ✅ Health endpoint: http://localhost:8000/health
- ✅ ASX trades executing (P&L tracking active)
- ✅ DynamoDB connectivity confirmed
- ✅ Both containers stable and healthy

## Troubleshooting

**Build fails:**
- Check Docker is running: `docker --version`
- Clear cache: `docker-compose build --no-cache`

**Container exits immediately:**
- Check logs: `docker-compose logs asx-trading`
- Verify AWS credentials in `.env`

**CRM API not accessible:**
- Check port mapping: `docker-compose ps`
- Verify container health: `curl http://localhost:8000/health`

**DynamoDB connection fails:**
- Verify AWS credentials: `docker-compose exec crm-api env | grep AWS`
- Check IAM permissions for DynamoDB access

---

**Generated**: 2025-10-28
**Status**: ✅ Local deployment successful → Ready for EC2 deployment
**Tested**: 2025-10-28 20:30 AEDT
