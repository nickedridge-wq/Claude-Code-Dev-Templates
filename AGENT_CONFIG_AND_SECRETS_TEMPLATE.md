# Agent Configuration & Secrets

---
Agent Name: [Name]
Version: [X.X]
Author: [Name]
Risk Level: [Low / Medium / High]
References: PROJECT_INSTRUCTIONS.md, CLAUDE_CODE_PROTOCOL.md
---

## Environment Variables / Secrets

| Name | Required? | Purpose | Default / Notes |
|------|-----------|---------|----------------|
| API_KEY | Yes | Access external service | User must supply |
| TIMEOUT | No | Operation timeout | 30s |
| FEATURE_FLAG | No | Toggle beta features | False |

## Optional Flags

- [Flag 1 – description, default]
- [Flag 2 – description, default]

## Runtime Limits

- Max retries: [Number]
- Max concurrent tasks: [Number]

## User Instructions

- Supply secrets via environment variables or secure config files
- Never commit secrets into version control
