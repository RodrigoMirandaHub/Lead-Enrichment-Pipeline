# Lead Enrichment Pipeline N8N + Groq + Supabase + HubSpot

Automated lead qualification pipeline that receives leads via webhook, scores them with an AI model, stores results in a Supabase database, creates contacts in HubSpot CRM, and fires email alerts for high-value leads.

## Architecture

```
Webhook (POST /lead)
    → Groq / LLaMA 3.3 — AI scoring
    → Code node — JSON parsing
    → Supabase — INSERT into leads table (duplicate protection)
    → IF score ≥ 7
        → HubSpot — Create or Update Contact
        → Gmail — Hot lead alert
```

## Features

- Webhook intake for lead data (name, email, company)
- AI-powered lead scoring (1–10) with category and reasoning via Groq LLaMA 3.3-70b
- Persistent storage in Supabase (PostgreSQL) with Row Level Security enabled
- Duplicate protection via unique constraint on email field
- Automatic contact creation in HubSpot CRM for hot leads
- Conditional Gmail alert for hot leads (score ≥ 7) with full lead details
- Full audit trail with timestamps in the database

## Tech Stack

| Layer | Tool |
|---|---|
| Automation | N8N |
| AI / LLM | Groq API — LLaMA 3.3-70b-versatile |
| Database | Supabase (PostgreSQL) |
| CRM | HubSpot |
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

-- Prevent duplicate leads
ALTER TABLE leads ADD CONSTRAINT leads_email_unique UNIQUE (email);
```

Row Level Security is enabled. The workflow uses the `service_role` key for authenticated inserts.

## Webhook Payload

Send a POST request to `/webhook/lead` with the following body:

```json
{
  "name": "Pedro Costa",
  "email": "pedro@microsoft.com",
  "company": "Microsoft"
}
```

## AI Scoring Output

The Groq node returns structured JSON:

```json
{
  "score": 8,
  "category": "hot",
  "reasoning": "The lead is from a well-known company and has a professional email address."
}
```

Categories: `hot` · `warm` · `cold`

## Alert Logic

If `score >= 7`:
- Contact is automatically created or updated in HubSpot CRM
- Gmail alert is sent with full lead details including name, email, company, score, category, and reasoning

## Gmail Alert Example

```
New hot lead detected!

Name: Pedro Costa
Email: pedro@microsoft.com
Company: Microsoft
Score: 8/10
Category: hot
Reasoning: The lead is from a well-known company and has a professional email address.
```

## Setup

1. Import the workflow JSON into your N8N instance
2. Create a Supabase project and run the schema SQL above
3. Create a HubSpot Service Key with `crm.objects.contacts.read` and `crm.objects.contacts.write` scopes
4. Add credentials in N8N: Groq API key (Header Auth), Supabase (Host + Service Role Secret), HubSpot (Service), Gmail (OAuth2)
5. Activate the workflow
6. Send a POST request to the production webhook URL to test

## Use Case

Built as a portfolio project demonstrating end-to-end automation with AI enrichment, database persistence, CRM integration, and real-time alerting — directly applicable to sales ops, lead generation, and growth engineering workflows.
