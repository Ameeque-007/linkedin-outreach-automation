# LinkedIn Outreach Automation

A three-stage n8n pipeline that scrapes LinkedIn leads from a search URL,
sends connection requests via PhantomBuster, and uses an LLM to draft
personalized outreach messages grounded in each prospect's actual profile
data — logging everything to Google Sheets along the way.

## How it works

The pipeline is split into three workflows, chained together by
PhantomBuster completion webhooks:

### 1. `01-scrapper.json` — Scrapper
A form collects a LinkedIn search URL and a session cookie. The session
cookie is stored in an n8n Data Table for the other workflows to reuse,
then a PhantomBuster agent is queued to scrape matching LinkedIn profiles.

### 2. `02-get-scrapped-leads.json` — Get Scrapped Leads
Triggered when the scrape completes. Reads the stored session cookie,
pulls the scraped profile URLs, logs them to a Google Sheet, then queues a
second PhantomBuster agent to send LinkedIn connection requests to those
leads.

### 3. `03-personalized-outreach.json` — Personalized Outreach
Triggered when the connection-request agent completes. Pulls enriched
profile data (name, title, headline, company, summary) for each processed
lead and sends it to an LLM (GPT-4.1-mini) with strict instructions to
generate three outreach messages — a connection note, a 1st follow-up, and
a 2nd follow-up — using only facts present in the actual profile data, no
invented details. Parses and validates the LLM's JSON output, then writes
the messages back to the Google Sheet alongside the lead record.

## Tools & APIs

- [n8n](https://n8n.io) — workflow orchestration
- [PhantomBuster](https://phantombuster.com) — LinkedIn scraping and
  connection-request automation
- Google Sheets API — lead logging and outreach message storage
- n8n Data Table — temporary session cookie storage between workflows
- OpenAI (via n8n's LangChain node, GPT-4.1-mini) — personalized message
  generation

## Setup

These exports have all credentials, API keys, resource IDs (spreadsheet/
data table), and PhantomBuster agent/identity IDs removed. To run this
yourself:

1. Import all three JSON files into your n8n instance.
2. Create your own PhantomBuster, Google Sheets, and OpenAI credentials in
   n8n, and replace the `YOUR_..._HERE` placeholders in each workflow with
   your own resource IDs (spreadsheet, data table, PhantomBuster agent and
   identity).
3. In PhantomBuster, configure the scraping agent to call
   `02-get-scrapped-leads.json`'s webhook on completion, and the
   connection-request agent to call `03-personalized-outreach.json`'s
   webhook on completion.
4. Activate all three workflows and submit a search URL through the form
   trigger in `01-scrapper.json`.
