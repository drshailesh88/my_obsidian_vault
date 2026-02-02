
# OpenClaw Bot Migration to GCP Cloud Run

## Complete Guide: VPS to Google Cloud Platform

---

## Table of Contents

1. [What We Started With](#what-we-started-with)
2. [What We Built](#what-we-built)
3. [Key Concepts](#key-concepts)
4. [Step-by-Step Migration](#step-by-step-migration)
5. [Problems We Faced & Solutions](#problems-we-faced--solutions)
6. [Final Working Configuration](#final-working-configuration)
7. [How to Deploy a Second Bot](#how-to-deploy-a-second-bot)
8. [Cost Breakdown](#cost-breakdown)

---

## What We Started With

### On VPS (Hetzner)

```
/home/openclaw/
├── .openclaw/
│   ├── openclaw.json          # Main config
│   ├── agents/                 # Agent data, memories
│   ├── telegram/               # Telegram state
│   └── auth-profiles.json      # API keys
├── docker-compose.yml
└── PM2 running the bot
```

### The Bot (OpenClaw)

- Telegram bot gateway
- Connects to AI providers (ZAI/GLM model)
- Stores conversation history and memories
- Needs writable directories for state

---

## What We Built

### On GCP Cloud Run

```
Cloud Run Service: openclaw
├── Container: openclaw (from Artifact Registry)
├── Secrets (mounted to /etc/):
│   ├── openclaw-config → /etc/openclaw/openclaw.json
│   └── openclaw-auth-profiles → /etc/openclaw-auth/auth-profiles.json
├── In-Memory Volume:
│   └── /home/node/.openclaw/ (writable)
└── Startup Script: copies configs, then starts gateway
```

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    Google Cloud Platform                 │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────────────────┐ │
│  │ Secret Manager  │    │      Artifact Registry      │ │
│  │                 │    │                             │ │
│  │ • openclaw-config    │ • openclaw:latest (image)   │ │
│  │ • openclaw-auth │    │                             │ │
│  └────────┬────────┘    └──────────────┬──────────────┘ │
│           │                            │                │
│           ▼                            ▼                │
│  ┌─────────────────────────────────────────────────────┐│
│  │                   Cloud Run                         ││
│  │  ┌───────────────────────────────────────────────┐  ││
│  │  │              Container                        │  ││
│  │  │                                               │  ││
│  │  │  /etc/openclaw/          (secrets, read-only) │  ││
│  │  │  /home/node/.openclaw/   (in-memory, writable)│  ││
│  │  │                                               │  ││
│  │  │  Startup: copy configs → start gateway        │  ││
│  │  └───────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────┘│
│                          │                              │
│                          ▼                              │
│                   Port 18789 (HTTPS)                    │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │     Telegram API        │
              │   (getUpdates polling)  │
              └─────────────────────────┘
```

---

## Key Concepts

### 1. Stateless vs Stateful

|Type|Description|Example|
|---|---|---|
|**Stateless**|No data persists between requests|API that calculates something|
|**Stateful**|Needs to remember things|Bot with conversation history|

**OpenClaw is stateful** - it needs to store memories, conversation history, telegram state.

**Cloud Run is designed for stateless** - containers can be killed anytime.

**Our solution:** In-memory volume (data survives during session, lost on restart)

### 2. Secret Management

```
WRONG: Hardcode secrets in code
WRONG: Pass secrets as build args
RIGHT: Use Secret Manager + mount at runtime
```

Secrets are mounted as files, not baked into the image.

### 3. The Permission Problem

Cloud Run runs containers as non-root user. When you mount a secret:

```
/home/node/.openclaw/openclaw.json  ← mounted by Cloud Run
/home/node/.openclaw/               ← directory becomes ROOT-owned, read-only
```

The app can’t create subdirectories like `/home/node/.openclaw/agents/`.

**Solution:** Mount secrets elsewhere, use in-memory volume for app directory, copy at startup.

### 4. Telegram Long Polling Conflicts

Telegram only allows ONE connection per bot token polling for updates.

```
Instance A: getUpdates → ✓
Instance B: getUpdates → 409 Conflict!
```

**Causes:** - Old VPS still running - Multiple Cloud Run revisions during deployment - Local development instance

**Solution:** 1. Stop all other instances 2. Call Telegram `close` API to reset sessions 3. Wait for rate limits if needed

---

## Step-by-Step Migration

### Step 1: GCP Project Setup

```
# Authenticate with service account
gcloud auth activate-service-account --key-file=/path/to/key.json

# Set project
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable \
  run.googleapis.com \
  artifactregistry.googleapis.com \
  secretmanager.googleapis.com \
  cloudbuild.googleapis.com
```

### Step 2: Create Artifact Registry Repository

```
gcloud artifacts repositories create openclaw-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="OpenClaw container images"
```

### Step 3: Build and Push Docker Image

```
cd /path/to/openclaw

# Build and push using Cloud Build
gcloud builds submit \
  --tag us-central1-docker.pkg.dev/PROJECT_ID/openclaw-repo/openclaw:latest
```

### Step 4: Create Secrets

```
# Main config
gcloud secrets create openclaw-config \
  --data-file=openclaw-config.json

# Auth profiles (API keys)
gcloud secrets create openclaw-auth-profiles \
  --data-file=auth-profiles.json

# Grant Cloud Run access
gcloud secrets add-iam-policy-binding openclaw-config \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

gcloud secrets add-iam-policy-binding openclaw-auth-profiles \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### Step 5: Deploy to Cloud Run

```
gcloud run deploy openclaw \
  --region=us-central1 \
  --image=us-central1-docker.pkg.dev/PROJECT_ID/openclaw-repo/openclaw:latest \
  --set-secrets="/etc/openclaw/openclaw.json=openclaw-config:latest,/etc/openclaw-auth/auth-profiles.json=openclaw-auth-profiles:latest" \
  --set-env-vars="ZAI_API_KEY=your-api-key-here" \
  --add-volume=name=openclaw-home,type=in-memory,size-limit=1Gi \
  --add-volume-mount=volume=openclaw-home,mount-path=/home/node/.openclaw \
  --port=18789 \
  --cpu=2 \
  --memory=2Gi \
  --min-instances=1 \
  --max-instances=1 \
  --allow-unauthenticated \
  --concurrency=160 \
  --timeout=300 \
  --cpu-boost \
  --command="sh" \
  --args="-c,mkdir -p /home/node/.openclaw/agents/main/agent && cat /etc/openclaw/openclaw.json > /home/node/.openclaw/openclaw.json && cat /etc/openclaw-auth/auth-profiles.json > /home/node/.openclaw/agents/main/agent/auth-profiles.json && exec node dist/index.js gateway --bind lan --port 18789"
```

### Step 6: Stop Old Instances

```
# On VPS
ssh root@YOUR_VPS_IP "pkill -f PM2 && pkill -f node"

# Close Telegram sessions
curl "https://api.telegram.org/botYOUR_BOT_TOKEN/close"
```

### Step 7: Verify

```
# Check logs
gcloud run services logs read openclaw --region=us-central1 --limit=50

# Look for:
# - [gateway] listening on ws://0.0.0.0:18789
# - [telegram] [primary] starting provider
# - NO "EACCES permission denied" errors
# - NO "getUpdates conflict" errors (after closing sessions)
```

---

## Problems We Faced & Solutions

### Problem 1: Container Failed to Start

**Error:** `The user-provided container failed to start and listen on the port`

**Cause:** Wrong port, missing config, or startup crash

**Solution:** - Ensure `--port` matches what app listens on - Check logs for actual error - Verify config files exist

---

### Problem 2: Permission Denied (EACCES)

**Error:** `EACCES: permission denied, open '/home/node/.openclaw/agents/main/agent/auth-profiles.json'`

**Cause:** Secret mounted in directory makes it read-only

**Solution:**

```
# Mount secrets to /etc/ (separate location)
--set-secrets="/etc/openclaw/openclaw.json=openclaw-config:latest"

# Make app directory writable with in-memory volume
--add-volume=name=openclaw-home,type=in-memory,size-limit=1Gi
--add-volume-mount=volume=openclaw-home,mount-path=/home/node/.openclaw

# Copy configs at startup
--command="sh"
--args="-c,mkdir -p /home/node/.openclaw/agents/main/agent && cat /etc/openclaw/openclaw.json > /home/node/.openclaw/openclaw.json && cat /etc/openclaw-auth/auth-profiles.json > /home/node/.openclaw/agents/main/agent/auth-profiles.json && exec node dist/index.js gateway --bind lan --port 18789"
```

---

### Problem 3: Telegram 409 Conflict

**Error:** `getUpdates conflict: terminated by other getUpdates request`

**Cause:** Multiple instances polling same bot token

**Solution:** 1. Find and stop all other instances: ```bash # Check VPS ssh root@VPS_IP “ps aux | grep node”

# Check local machine ps aux | grep openclaw

# Check Cloud Run revisions gcloud run revisions list –service=openclaw –region=us-central1 ```

2. Close Telegram sessions:
    
    ```
    curl "https://api.telegram.org/botYOUR_TOKEN/close"
    ```
    
3. If rate limited, wait (error shows `retry_after` in seconds)
    

---

### Problem 4: Pairing Required

**Error:** `OpenClaw: access not configured. Pairing code: XXXXX`

**Cause:** `dmPolicy: "pairing"` requires manual approval

**Solution:** Change to `dmPolicy: "allowlist"` and add user IDs:

```
{
  "channels": {
    "telegram": {
      "dmPolicy": "allowlist",
      "allowFrom": ["YOUR_TELEGRAM_USER_ID"],
      "accounts": {
        "primary": {
          "dmPolicy": "allowlist",
          "allowFrom": ["YOUR_TELEGRAM_USER_ID"]
        }
      }
    }
  }
}
```

---

### Problem 5: Token Mismatch on Dashboard

**Error:** `unauthorized: gateway token mismatch`

**Cause:** Browser cached old token

**Solution:** Use tokenized URL:

```
https://YOUR_CLOUD_RUN_URL/?token=YOUR_GATEWAY_TOKEN
```

---

## Final Working Configuration

### openclaw-config.json

```
{
  "auth": {
    "profiles": {
      "zai:default": {
        "provider": "zai",
        "mode": "api_key"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "zai/glm-4.7"
      }
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["YOUR_USER_ID"],
      "accounts": {
        "primary": {
          "name": "Your Bot",
          "dmPolicy": "allowlist",
          "botToken": "YOUR_BOT_TOKEN",
          "allowFrom": ["YOUR_USER_ID"]
        }
      }
    }
  },
  "gateway": {
    "port": 18789,
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "your-secret-token"
    },
    "controlUi": {
      "allowInsecureAuth": true
    },
    "trustedProxies": ["0.0.0.0/0"]
  }
}
```

### auth-profiles.json

```
{
  "version": 1,
  "profiles": {
    "zai:default": {
      "type": "api_key",
      "provider": "zai",
      "key": "YOUR_ZAI_API_KEY"
    }
  }
}
```

---

## How to Deploy a Second Bot

To avoid conflicts when deploying multiple bots:

### 1. Use Different Service Name

```
gcloud run deploy openclaw-bot-2 \  # Different name
  --region=us-central1 \
  ...
```

### 2. Use Different Secrets

```
gcloud secrets create openclaw-bot-2-config --data-file=bot2-config.json
gcloud secrets create openclaw-bot-2-auth --data-file=bot2-auth.json
```

### 3. Use Different Telegram Bot Tokens

Each bot needs its own token from @BotFather

### 4. Use Different Ports (if needed)

Cloud Run handles this automatically with different services

### 5. Complete Command for Second Bot

```
gcloud run deploy openclaw-bot-2 \
  --region=us-central1 \
  --image=us-central1-docker.pkg.dev/PROJECT_ID/openclaw-repo/openclaw:latest \
  --set-secrets="/etc/openclaw/openclaw.json=openclaw-bot-2-config:latest,/etc/openclaw-auth/auth-profiles.json=openclaw-bot-2-auth:latest" \
  --add-volume=name=openclaw-home,type=in-memory,size-limit=1Gi \
  --add-volume-mount=volume=openclaw-home,mount-path=/home/node/.openclaw \
  --port=18789 \
  --cpu=2 \
  --memory=2Gi \
  --min-instances=1 \
  --max-instances=1 \
  --allow-unauthenticated \
  --command="sh" \
  --args="-c,mkdir -p /home/node/.openclaw/agents/main/agent && cat /etc/openclaw/openclaw.json > /home/node/.openclaw/openclaw.json && cat /etc/openclaw-auth/auth-profiles.json > /home/node/.openclaw/agents/main/agent/auth-profiles.json && exec node dist/index.js gateway --bind lan --port 18789"
```

---

## Cost Breakdown

### Cloud Run Pricing (us-central1)

|Resource|Price|Our Usage|Monthly Cost|
|---|---|---|---|
|vCPU|$0.024/hr|2 vCPU × 24hr × 30 days|~$35|
|Memory|$0.0025/hr per GB|2GB × 24hr × 30 days|~$3.60|
|**Total**|||**~$38-40/month**|

### With min-instances=1

Bot is always running = always paying. But no cold starts.

### To Reduce Cost

```
--min-instances=0  # Scale to zero when idle
--cpu-throttling   # Reduce CPU when idle
```

But: Cold starts mean delayed responses.

---

## Important Notes

### Data Persistence

**WARNING:** In-memory volumes are ephemeral! - Data survives container restarts within same instance - Data is LOST when instance is replaced - For persistent data, need Cloud Storage or database

### For Important Bots with Memories

Consider: 1. **Compute Engine** - persistent disk, like VPS 2. **Cloud Storage** - mount bucket for data 3. **Cloud SQL** - if bot supports external database

---

## Quick Reference Commands

```
# View logs
gcloud run services logs read openclaw --region=us-central1 --limit=50

# Update secret
gcloud secrets versions add openclaw-config --data-file=openclaw-config.json

# Redeploy with new secret
gcloud run services update openclaw --region=us-central1 \
  --update-secrets="/etc/openclaw/openclaw.json=openclaw-config:latest"

# Check revisions
gcloud run revisions list --service=openclaw --region=us-central1

# Close Telegram session (if conflicts)
curl "https://api.telegram.org/botYOUR_TOKEN/close"

# Check service URL
gcloud run services describe openclaw --region=us-central1 --format="value(status.url)"
```

---

## Lessons Learned

1. **Cloud Run is for stateless apps** - fighting it for stateful apps is painful
2. **Permission issues** - Cloud Run’s security model conflicts with apps expecting writable home directories
3. **Telegram conflicts** - always stop old instances before deploying new ones
4. **Cost isn’t always lower** - managed services trade money for convenience
5. **VPS isn’t bad** - for personal infrastructure, a secured VPS is often simpler

---

_Document created: 2026-02-02_ _Migration completed for: OpenClaw Bot (sacrificable instance)_