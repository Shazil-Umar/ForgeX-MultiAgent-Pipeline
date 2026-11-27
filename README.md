# Forgex Multi-Agent B2B Outreach Pipeline

An automated, 4-stage lead ingestion, AI qualification, contact enrichment, and cold outreach pipeline built with **n8n**, **Supabase**, **Google Gemini**, **Apollo.io**, and **Gmail**.

---

## Architecture Overview

The system decouples lead generation into four independent n8n workflows coordinated through state transitions in a central Supabase PostgreSQL database (`jobs` table):

```text
[Remotive API] ──▶ ForgexAgent1 (Ingest) ──▶ Supabase (status: NEW)
                                                   │
[Gemini Flash] ◀── ForgexAgent2 (Score)  ◀─────────┘
      │
      ▼
Supabase (status: Processed, ai_score > 6)
      │
      ▼
ForgexAgent3 (Apollo Lookup + Gemini Copywriter) ──▶ Supabase (status: READY_TO_SEND)
                                                              │
[Gmail Dispatch] ◀── ForgexAgent4 (Outreach) ◀────────────────┘
      │
      ▼
Supabase (status: Drafted)
```

### Workflow Breakdown

| Workflow File | Role | Description |
| :--- | :--- | :--- |
| `workflows/ForgexAgent1.json` | **Job Ingestion** | Queries the Remotive API (`limit=100`), splits the `jobs` payload into individual records, and inserts new postings into Supabase with `status = "NEW"`. |
| `workflows/ForgexAgent2.json` | **AI Qualification & Scoring** | Fetches `NEW` jobs, evaluates company size, remote eligibility, staffing-agency exclusions, and tech stack relevance via Gemini (`jsonOutput: true`), and writes `ai_score`, `ai_summary`, and `tech_stack` back to Supabase (`60s` rate-limit pacing). |
| `workflows/ForgexAgent3.json` | **Enrichment & Drafting** | Pulls qualified leads (`ai_score > 6` and `status = "PROCESSED"`), queries Apollo's `/mixed_people/api_search` endpoint for engineering/hiring decision-makers, generates a tailored <150-word email draft via Gemini, and marks rows `READY_TO_SEND` (`65s` pacing). |
| `workflows/ForgexAgent4.json` | **Outreach Dispatch** | Fetches `READY_TO_SEND` records, dispatches emails via Gmail OAuth2, and updates each record's status to `"Drafted"`. |

---

## Supabase Schema (`jobs` Table)

To run these workflows in your own n8n instance, create a `jobs` table in Supabase with the following columns:

```sql
create table public.jobs (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz default now(),
  job_title text,
  company_name text,
  job_url text unique,
  job_description text,
  technical_category text,
  location text,
  remote_status text,
  source text,
  status text default 'NEW',
  ai_score integer,
  ai_summary text,
  tech_stack text,
  contact_name text,
  contact_email text,
  contact_linkedin text,
  outreach_draft text
);
```

---

## Setup & Configuration

1. **Import Workflows:** In n8n, open **Workflows → Import from File** and import `ForgexAgent1.json` through `ForgexAgent4.json` from the `workflows/` directory.
2. **Configure Credentials:**
   * **Supabase API:** Attach your Supabase project URL and `service_role` / `anon` key to the Supabase nodes across all four workflows.
   * **Google Gemini (PaLM) API:** Configure your Gemini API credential on the `Message a model` nodes in `ForgexAgent2` and `ForgexAgent3`.
   * **Apollo.io API:** In `ForgexAgent3` (`HTTP Request` node), replace `YOUR_APOLLO_API_KEY` in the `X-Api-Key` header (or move it to an n8n Header Auth credential).
   * **Gmail OAuth2:** Connect your Gmail account on the `Send a message` node in `ForgexAgent4`.

---

## Current Status & Known Issues (WIP)

This repository preserves the working prototype architecture along with a few known bugs identified prior to pausing development:

* **Status Casing Mismatch (`ForgexAgent2` → `ForgexAgent3`):** `ForgexAgent2` updates rows to `"Processed"`, while `ForgexAgent3` queries for `status = "PROCESSED"` (case-sensitive in PostgreSQL).
* **Expression Syntax in `ForgexAgent3` (`Update a row`):**
  * `contact_name` expression is missing a closing `}}` (`={{ $('HTTP Request').item.json.people[0].name`).
  * `contact_email` and `contact_linkedin` field IDs contain a leading `=` prefix (`=contact_email`).
  * Duplicate `outreach_draft` field mappings exist in the update node (`$json.text` and `$json.content.parts[0].text`).
* **Array Indexing in `ForgexAgent1` (`Create a row`):** `company_name` references `jobs[0].company_name` instead of the current split item (`$json.company_name`).
* **Duplicate Prompt Sections in `ForgexAgent2`:** The system prompt contains duplicated `HARD REJECTION RULES` and `SCORING SYSTEM` blocks that should be consolidated.
* **Apollo Domain Query (`ForgexAgent3`):** `q_organization_domains` currently passes `company_name` rather than the extracted `company_domain` from `ForgexAgent2`.