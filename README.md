# Lead Enrichment Pipeline — N8N + Groq + Supabase

Automated lead qualification pipeline that receives leads via webhook, scores them with an LLM, stores results in a Supabase database, and fires email alerts for high-value leads.

<img width="1668" height="741" alt="Screenshot_31" src="https://github.com/user-attachments/assets/fc83d854-60be-4161-bd4f-2e7b775260b6" />


## Architecture

```
Webhook (POST /lead)
    → Groq / LLaMA 3.3 — AI scoring
    → Code node — JSON parsing
    → Supabase — INSERT into leads table
    → IF score ≥ 7 → Gmail alert
```

## Features

- Webhook intake for lead data (name, email, company)
- AI-powered lead scoring (1–10) with category and reasoning via Groq LLaMA 3.3-70b
- Persistent storage in Supabase (PostgreSQL) with Row Level Security enabled
- Conditional Gmail alert for hot leads (score ≥ 7)
- Full audit trail with timestamps in the database

## Tech Stack

| Layer | Tool |
|---|---|
| Automation | N8N |
| AI / LLM | Groq API — LLaMA 3.3-70b-versatile |
| Database | Supabase (PostgreSQL) |
| Alerts | Gmail via N8N node |
| Language | JavaScript (N8N Code node) |

## Database Schema

```sql
CREATE TABLE leads (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  name text,
  email text,
  company text,
  score integer,
  category text,
  reasoning text,
  created_at timestamp DEFAULT now()
);
```

Row Level Security is enabled. The workflow uses the `service_role` key for authenticated inserts.

## Webhook Payload

Send a POST request to `/webhook/lead` with the following body:

```json
{
  "name": "João Silva",
  "email": "joao@empresa.com",
  "company": "TechCorp Brasil"
}
```

## AI Scoring Output

The Groq node returns structured JSON:

```json
{
  "score": 8,
  "category": "warm",
  "reasoning": "The lead has a professional email address and is associated with a company, indicating potential legitimacy and interest."
}
```

Categories: `hot` · `warm` · `cold`

## Alert Logic

If `score >= 7`, a Gmail alert is sent with the full lead details including name, email, company, score, category, and reasoning.

## Setup

1. Import the workflow JSON into your N8N instance
2. Create a Supabase project and run the schema SQL above
3. Add credentials in N8N: Groq API key (Header Auth), Supabase (Host + Service Role Secret), Gmail OAuth
4. Activate the workflow
5. Send a POST request to the production webhook URL to test

## Use Case

Built as a portfolio project demonstrating end-to-end automation with AI enrichment and database persistence — directly applicable to CRM lead qualification, sales ops, and growth engineering workflows.
