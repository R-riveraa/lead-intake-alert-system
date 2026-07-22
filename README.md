# Lead Intake & Agent Alert System

A lead intake and qualification pipeline built for a real estate lead-response use case. Built twice — once in n8n, once in Zapier — to validate the same core logic across two automation platforms.

**[Watch the demo](https://www.loom.com/share/610692eabe2c451dadf8c055ee794a94)** — a lead submits the form, gets scored, logged, and triggers an SMS alert in 30 seconds.


## What it does

1. A lead submits contact info and a message through a hosted intake form (n8n Form Trigger)
2. The phone number is validated via a direct REST API call (Veriphone) before any other processing happens
3. An LLM (Gemini) scores the lead's urgency from 1-10 based on message content and stated timeframe
4. Leads scoring above a configurable threshold are flagged high-priority; others are marked standard
5. Every lead is logged into Airtable with a real CRM-style pipeline stage (New → Contacted → Qualified)
6. High-priority leads with a validated phone number trigger an SMS alert via Twilio
7. **n8n only:** leads missing required contact info (name or phone) are logged with an Error status instead of failing silently or being discarded

See [architecture-diagram.mermaid](./architecture-diagram.mermaid) for a simple visual.

## Stack

- **n8n** (primary build) — webhook trigger, conditional branching, native AI node integration
- **Zapier** (parallel rebuild) — same logic using Paths for branching, Formatter for data extraction
- **Airtable** — structured CRM-style data store with a separate Settings table for configurable scoring rules
- **Twilio** — SMS alerting
- **Veriphone** — phone number validation via direct HTTP request (not a native app connector)
- **Google Gemini** — AI-based urgency classification

## Design notes

- **Configurable scoring threshold**: rather than hardcoding a priority cutoff, the n8n build reads the threshold from a dedicated Airtable Settings table. This means the same automation can be tuned per client without touching the underlying workflow logic.
- **Minimal required fields**: the system only hard-requires name and phone number to accept a lead. Email, message, and timeframe are useful for scoring but not essential for follow-up — a lead missing those can still be contacted. This keeps the intake broad by default; additional required fields are easy to add if a client's process demands stricter intake.
- **Phone validation before AI scoring**: validating contact info early avoids wasting an AI call and avoids alerting on unreachable leads.
- **Why a failed lead still gets logged**: a lead with no name or phone number can't realistically be followed up on. The point of logging it as an error isn't to queue it for someone to complete later — it's so a broken form, a bad integration, or bot traffic hitting the webhook shows up somewhere visible instead of just disappearing. If errors start piling up, that's a signal something upstream needs attention.
- **Known scope difference between builds**: the n8n version includes the missing-field error-log path described above. The Zapier version does not — a lead missing name/phone currently has no matching Path condition and the run ends without producing any record. This was a deliberate time-boxing decision for the parallel rebuild, not an oversight, but it's a real gap that would need closing before either version went to a live client.

## What's intentionally left out of this repo

- The exact AI classification prompt
- The full exported n8n workflow JSON / Zapier blueprint
- API keys and credentials (used locally only, never committed)

These are the most easily-copyable, highest-leverage parts of the build and are kept private for client work.

## Demo

See the accompanying Loom recording for a live end-to-end walkthrough.
