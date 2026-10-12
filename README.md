# Enterprise RAG Platform

> A private AI assistant that answers employees from their company's own documents, built solo and shipped to production during my internship.

**Internship · GS Group (video-surveillance & security, ~15 people) · Summer 2026**
Source code belongs to the company and stays private. Happy to walk you through the architecture or demo it → **imran.nogueira@gmail.com**

## Overview

A retrieval-augmented assistant for the Support, Legal and Sales departments, answering from the company's real internal documents (manuals, contracts, quotes, invoices) with strict isolation between departments. I designed, built and deployed it end to end; it runs in production on the company's own server and was handed over to the team at the end of the internship.

## What it does

- **Grounded answers** — retrieval over internal documents; the model is instructed to answer *only* from the retrieved excerpts. Web search is a last resort and always shows the date of the information. Every answer carries a source badge (internal / web / document analysis / attached file).
- **Per-department isolation** — each account only sees the departments it is allowed to; each department lives in its own vector collection, so data never leaks across teams.
- **Cross-document intelligence** — compare an invoice with a quote, summarize a customer, track unbilled quotes, with deterministic calculations instead of LLM arithmetic.
- **Document handling** — in-chat PDF preview, linked documents (same customer / reference), scanned and handwritten documents transcribed by vision, files attached to a single conversation without entering the shared base.
- **Ready-to-send emails** — generates an `.eml` with the document attached, opens in Mail / Outlook.
- **Administration** — user / admin / super-admin roles, rights management, drag-and-drop ingestion with automatic routing to the right department.

## Security work

Shipped on the last day of the internship:
- **2FA (TOTP)** with an authenticator app, two-step login.
- **Invitation links** — signed tokens, the user sets their own password.
- **Machine-to-machine API keys** for third-party software (SHA-256 hashed, `X-API-Key` header).

Plus JWT sessions, bcrypt hashing, company-domain-only accounts, and no secret ever reaching the frontend.

## Stack

| Layer | Tech |
|---|---|
| Backend | Python, FastAPI (layered: config / core / routers / deps), psycopg pool |
| Frontend | React + Vite, in-house CSS design system, single API client layer |
| Data | PostgreSQL (accounts, rights, history), ChromaDB (vectors), files on disk |
| AI | OpenAI — `gpt-4o-mini` for chat/RAG, `gpt-4o` for vision, `text-embedding-3-small` |
| Infra | Docker Compose, Caddy (automatic HTTPS), OVH VPS (Ubuntu) |

## How it works

```
Employee ──HTTPS──▶ Caddy ──▶ React frontend ──/api──▶ FastAPI
                                                     ├─▶ PostgreSQL  (accounts, rights, history)
                                                     ├─▶ ChromaDB    (one collection per department)
                                                     └─▶ OpenAI      (embeddings, LLM, vision)
```

Question → embedding → top-5 excerpts from the departments the user is allowed to see → answer generated from those excerpts only → source badge.

## Key decisions

- **Migrated from Supabase to self-hosted** PostgreSQL + ChromaDB — the company keeps its documents on its own server, and the running cost drops.
- **One vector collection per department** — isolation is structural: a user's query only ever reaches the collections of the departments they are allowed to see.
- **Two model tiers** — the small model for everyday work, the large one only where it matters (vision), to keep token costs under control.

## Results

- Deployed in production on the company's domain.
- Full handover: deployment, security and API integration docs, code and server access transferred.
- Internship assessed by Epitech (report + oral defense): **top mark at the oral**.

## Contact

Architecture walkthrough or live demo on request → **imran.nogueira@gmail.com** · [Portfolio](https://imran-nogueira.github.io)
