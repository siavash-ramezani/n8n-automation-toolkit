# n8n Automation Toolkit

A curated collection of production-style [n8n](https://n8n.io) workflows demonstrating automation and integration skills — form/lead handling, notifications, error-tolerant API calls, and other patterns commonly needed in real business automation. Each workflow is documented, exported as importable JSON, and built with credentials kept out of the repo (environment variables and placeholder URLs only).

## Workflows

| Workflow | Description | Status |
|---|---|---|
| [Lead Capture](workflows/lead-capture/README.md) | Webhook-triggered intake that validates form data, pushes leads to a CRM, and notifies a Slack channel — with fallback notification if the CRM call fails | ✅ Available |
| [AI Content Moderation](workflows/ai-content-moderation/README.md) | LLM-classified first-pass review that auto-publishes approved content, routes flagged/rejected content to Slack for human follow-up, and fails safe to human review if the LLM call or parse fails | ✅ Available |
| Scheduled Report Generator | Generate and deliver a recurring report on a schedule | 🔜 Coming soon |
| Webhook Data Sync | Keep two systems in sync in response to incoming webhook events | 🔜 Coming soon |

## Repository structure

```
n8n-automation-toolkit/
├── workflows/
│   ├── lead-capture/
│   │   ├── workflow.json   # Importable n8n workflow export
│   │   └── README.md       # Workflow-specific documentation
│   └── ai-content-moderation/
│       ├── workflow.json   # Importable n8n workflow export
│       └── README.md       # Workflow-specific documentation
├── .env.example             # Environment variables referenced by workflows (placeholders only)
└── README.md                 # This file
```

## Credentials & safety

No workflow in this repo contains real API keys, tokens, or webhook URLs. Workflows reference environment variables (e.g. `CRM_API_KEY`, `SLACK_WEBHOOK_URL`) that you must configure in your own n8n instance. See each workflow's README for the specific variables it needs, and [.env.example](.env.example) for the full list.
