# CRM Lead Intake & Notification Automation

An end-to-end workflow automation built in **n8n** that captures form submissions, validates data, syncs contacts to **Salesforce**, and sends real-time notifications — with a dedicated error-handling branch and full execution logging.

> **Note on this project:** This is a self-directed learning project, not a production deployment. I built it to develop hands-on skills in REST APIs, webhooks, CRM integration, and workflow automation, ahead of applying for technical support / integration support roles. Every node shown here was built and tested by me in n8n.

---

## Why I built this

I wanted to move past reading API documentation and actually build something that behaves like a real support/integration workflow: taking in data, validating it, pushing it into a CRM, handling failures gracefully, and notifying the right people when something breaks. This project let me practice the exact skills technical support and integration engineers use daily — API authentication, troubleshooting failed requests, reading status codes, and reasoning about data mapping between systems.

---

## Architecture

```
Google Form
    │
    ▼
Google Sheets Trigger
    │
    ▼
Edit Fields (normalize/map data)
    │
    ▼
IF: Required fields present?
    │
    ├── YES ──▶ Salesforce Upsert Contact (match on Email)
    │               │
    │               ▼
    │           HTTP Request (supplementary call)
    │               │
    │               ▼
    │           Slack Notification (success)
    │               │
    │               ▼
    │           Gmail Confirmation Email
    │               │
    │               ▼
    │           Append Execution Log → Google Sheets
    │
    └── NO ───▶ Slack Notification (validation failed)
                    │
                    ▼
                Append Failed Log → Google Sheets

Global Error Workflow (attached to all nodes)
    │
    ▼
Slack Alert on any runtime failure
```

---

## What it does

1. **Trigger** – A new submission on a connected Google Sheet (fed by a Google Form) triggers the workflow.
2. **Normalize** – The `Edit Fields` node maps and cleans incoming form data into the structure the rest of the workflow expects.
3. **Validate** – An `IF` node checks that required fields are present before continuing.
4. **Sync to CRM** – On success, the record is upserted into **Salesforce** (create-or-update, matched by email) so duplicate submissions don't create duplicate contacts.
5. **Notify** – A Slack message confirms the successful sync, and a Gmail confirmation email is sent to the submitter.
6. **Log** – Every run (success or failure) is appended to a Google Sheets log for auditability.
7. **Handle failures** – Validation failures branch into their own Slack alert + failed-log entry. Separately, a **Global Error Workflow** catches any runtime failure across the pipeline (e.g. an API call failing) and sends a Slack alert immediately.

---

## Tech stack

| Category | Tools |
|---|---|
| Automation platform | [n8n](https://n8n.io) |
| CRM | Salesforce (REST API, via n8n's Salesforce node) |
| Notifications | Slack API, Gmail API |
| Data source | Google Forms, Google Sheets |
| API testing | Postman |
| Version control | Git |

---

## Skills demonstrated

- REST API concepts (endpoints, HTTP methods, status codes, headers)
- API authentication (OAuth basics)
- Webhook and trigger-based automation
- CRM data modeling and upsert logic
- Conditional branching / data validation
- Error handling and alerting design
- Execution logging for auditability
- Cross-platform integration (forms → sheets → CRM → chat → email)

---

## Repository structure

```
├── workflows/
│   ├── main-lead-intake-workflow.json   # Exported n8n workflow (main pipeline)
│   └── global-error-workflow.json       # Exported n8n error-handling workflow
├── docs/
│   ├── architecture-diagram.png
│   ├── screenshot-workflow-overview.png
│   ├── screenshot-salesforce-node-config.png
│   ├── screenshot-slack-notification.png
│   └── screenshot-execution-log.png
└── README.md
```

---

## Screenshots

<img width="1877" height="926" alt="image" src="https://github.com/user-attachments/assets/6c0d16c7-e3c7-4fb1-80ed-33d51e2da9aa" />
This is the workflow with the highlighted node to force fail to trigger the error handling trigger workflow

<img width="1707" height="917" alt="image" src="https://github.com/user-attachments/assets/d106a15d-53d1-40c3-9a18-59179e1c7421" />
Original workflow

<img width="1668" height="872" alt="image" src="https://github.com/user-attachments/assets/15ca0e7d-213d-4547-bee1-a20343277dd0" />
Error handling workflow

<img width="1856" height="537" alt="image" src="https://github.com/user-attachments/assets/c7a0c6b3-09ac-42a4-a385-d906443cc881" />
Automation log results




- Full workflow canvas (all nodes, both branches)
- Salesforce node configuration (upsert-on-email logic)
- A Slack alert triggered by the Global Error Workflow
- The Google Sheets execution log after a few test runs (success + failure rows)

---

## What I'd improve next

- Move credentials out of node configs into environment-based credentials (already partially done via n8n's credential store)
- Add retry logic with exponential backoff on the HTTP Request node
- Add a lightweight test harness to simulate malformed form submissions
- Expand logging to capture response payloads for easier debugging

---

## Contact

**Omar Shams**
omarelfekyy2@gmail.com | [LinkedIn](https://www.linkedin.com/in/omarelfekyy2/)
