# Multi-Region Dispatch Automation Snippets

A public, employer-safe overview of a multi-region operational dispatch automation in n8n. It covers email triggers, AI-assisted parsing, modular region workflows, human approval gates, asynchronous DocuSign signing, scheduled reminders, and Slack summaries across Microsoft 365, DocuSign, ClickSend, Slack, and Azure OpenAI.

This repository documents the architecture decisions and implementation approach from a dispatch automation I built and maintained in a company setting. The examples are rewritten for public review so the technical work can be discussed without exposing customer, employer, tenant, or workflow secrets.

> Customer data, secrets, tenant identifiers, internal SharePoint paths, DocuSign envelope IDs, and raw workflow exports are not part of this repository. The snippets are public-safe examples: field names are neutral, project-specific identifiers are removed, and comments are translated to English.

---

## What This Pattern Solves

An operations team receives structured dispatch emails from multiple regional offices. Each booking can trigger several downstream actions:

- Folder creation in SharePoint, partitioned by region and service type
- Tracking row creation in Excel for operations dashboards
- Digital signature workflow through DocuSign
- Confirmation SMS to the customer
- Day-before preparation SMS through a scheduled workflow
- Reminder SMS if the document remains unsigned after a defined threshold
- Slack summary at the end of the day
- Cross-region SharePoint upload when the signed PDF arrives back

Handled manually, this kind of process creates repeated handoffs, weak traceability, and avoidable follow-up work. The automated version keeps the familiar Microsoft 365 tools in place while surfacing only the rows that need a human decision.

---

## Architecture at a Glance

```
┌─────────────────── Layer 1: Trigger ────────────────────┐
│   Outlook Trigger (Region A)  ─┐                        │
│   Outlook Trigger (Region B)  ─┤── Switch ─► Pre-Dispatch│
│   Outlook Trigger (Region C)  ─┘            (per region) │
└──────────────────────────────────────────────────────────┘
                                                  │
┌─────────────── Layer 2: Pre-Dispatch ───────────▼────────┐
│   HTML strip → AI parse → sanitize → approval gate       │
│        → folder + Excel + DocuSign + SMS + Slack         │
│  (calls Layer 3 sub-workflows via executeWorkflow)       │
└──────────────────────────────────────────────────────────┘
                                                  │
┌─────────────── Layer 3: Sub-workflows ──────────▼────────┐
│  Synchronous (called from Layer 2):                      │
│    • Daily duty roster lookup                            │
│    • Project manager onboarding to master table          │
│    • DocuSign envelope creation                          │
│    • ClickSend SMS sending                               │
│    • PDF operations helper                               │
│                                                          │
│  Asynchronous (own triggers):                            │
│    • DocuSign completion → SharePoint upload             │
│    • Day-before SMS reminders (cron)                     │
│    • Daily summary + overdue escalation (cron)           │
└──────────────────────────────────────────────────────────┘
```

See [`docs/architecture.md`](docs/architecture.md) for the full system breakdown including the sub-workflow inventory.

---

## Repository Structure

```
.
├── README.md                          ← you are here
├── docs/
│   ├── architecture.md                ← full system architecture
│   ├── docusign-integration.md        ← envelope send + webhook completion
│   └── scheduled-workflows.md         ← cron + reminders + daily summary
└── snippets/
    ├── README.md                      ← snippet index
    ├── llm-output-sanitizer.js        ← clean LLM output before downstream
    ├── email-html-stripper.js         ← strip HTML from Outlook body
    ├── batch-deduplication.js         ← unique-key processing in loops
    ├── data-shape-transformer.js      ← raw spreadsheet → clean shape
    ├── binary-attachment-extractor.js ← fan out email attachments
    ├── appointment-date-filter.js     ← robust local-date "tomorrow" filter
    ├── docusign-envelope-extractor.js ← parse envelope ID from notification email
    ├── folder-path-sanitizer.js       ← safe SharePoint path generation
    └── scheduled-summary-aggregator.js← daily Slack summary builder
```

The snippets are adapted from real n8n Code-node work. They show implementation shape and reliability decisions without republishing raw workflow exports.

---

## Tech Stack

- **Orchestration:** n8n
- **Triggers:** Microsoft Outlook, Schedule/Cron, DocuSign notification flow
- **AI:** Azure OpenAI with structured output parsing
- **Storage:** Microsoft Excel via Microsoft Graph, Microsoft OneDrive, SharePoint
- **External APIs:** DocuSign eSignature, ClickSend SMS, Microsoft Graph
- **Notifications:** Slack App Home, DMs, and channel posts
- **Code:** JavaScript in n8n Code nodes

---

## Purpose

I use this repository to explain the technical structure behind a multi-step automation project: dispatch routing, approval gates, document signing, scheduled reminders, and Microsoft 365 integrations, while keeping operational data private.

The full portfolio case study is available at [valentinoveljanovski.de/projects/multi-region-dispatch-automation](https://valentinoveljanovski.de/projects/multi-region-dispatch-automation).

---

## About

Built and maintained by [Valentino Veljanovski](https://valentinoveljanovski.de), automation developer based in München.

---

## Viewing Notice

This repository is published for public review during hiring and collaboration conversations.

All code, documentation, diagrams, and content in this repository remain the intellectual property of the author. **All rights reserved.**

No license is granted, expressed or implied, for reuse, redistribution, modification, or commercial use of any material in this repository without prior written permission from the author.

For licensing or collaboration inquiries, contact: <valentinoveljanovski@outlook.com>
