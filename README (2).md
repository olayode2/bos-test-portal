# BOS Test Portal: Order Padi AI Sales Agent

A single-file web portal for onboarding and running **Order Padi**, a multi-tenant WhatsApp AI sales agent for small businesses. A business owner can set up their shop, connect their WhatsApp Business number, review what the AI knows, approve it for launch, and then monitor live chats, leads and orders, all from one page.

The portal is a pure front end (`index.html`, no build step, no framework). All data and AI logic live in n8n workflows exposed as webhooks. The portal talks to them over HTTPS/JSON.

## The five-step flow

| Step | Tab | What it does |
|------|-----|--------------|
| 1 | **Business** | Captures business name, type, tone, products and prices, delivery rules, hours, bank details and the owner's WhatsApp number. Validates required fields and saves per client |
| 2 | **WhatsApp** | Connects the business number through Meta's Embedded Signup (Facebook SDK login). A live checklist tracks authorisation, number verification and channel status |
| 3 | **AI Knowledge** | Shows the approved knowledge base with completeness stats. The owner adds FAQs, offers, escalation rules, qualification questions and a reorder-reminder interval |
| 4 | **Approval** | Pre-launch checklist, a test chat with the agent (same AI and rules, nothing sent on WhatsApp or saved), and a client approve / pause switch |
| 5 | **Live** | Dashboard with KPIs, setup status and four views: Chats, Leads, Orders and Human handoff |

## Features

- **Multi-client portal:** a dropdown switches between businesses, and `?client=<id>` deep-links straight to one
- **Guardrailed agent:** the AI never invents answers, only gives discounts that are approved offers, and automatically hands over to a human for refunds, complaints, exceptions, sensitive issues or requests to speak to a manager
- **Human takeover from the dashboard:** pause the AI on any chat, reply from the business WhatsApp number, then hand the chat back
- **Follow-up tracking:** flags leads due for follow-up, using a three-attempt follow-up cap
- **Safe test mode:** try the agent as a customer before going live
- **Auto-refreshing dashboard** (15-second polling), responsive layout for mobile
- **Output escaping** on all user-supplied text rendered into the page

## Architecture

```
Browser (index.html)
   │  fetch + JSON
   ▼
n8n webhooks (self-hosted)
   ├─ bos-portal-data      list clients / load dashboard data
   ├─ bos-portal-brand     save business and knowledge settings
   ├─ bos-portal-connect   complete WhatsApp (Meta Embedded Signup) connection
   ├─ bos-portal-action    approve / pause agent, take over, reply, resume chat
   └─ bos-agent-test       sandboxed test conversation with the AI agent
```

The WhatsApp Cloud API, the LLM calls and the database sit behind those webhooks, so the browser holds no secrets. The Meta App ID and Config ID in the file are public identifiers, not credentials.

## Run it

It's one static file, so there's nothing to install:

```bash
git clone https://github.com/olayode2/bos-test-portal
cd bos-test-portal
python3 -m http.server 8080   # then open http://localhost:8080
```

To point it at your own backend, change the `API` constant near the top of the `<script>` block. WhatsApp connection requires the page to be served over HTTPS on a domain allowed in your Meta app settings.

## Stack

HTML · CSS · vanilla JavaScript · Meta Embedded Signup (Facebook JS SDK) · WhatsApp Business Platform · n8n webhooks
