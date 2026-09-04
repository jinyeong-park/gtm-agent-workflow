# Autonomous B2B GTM Lead Generation & Outreach Agent

An end-to-end, multi-stage agentic workflow built using **[Claude Code](https://docs.anthropic.com/en/docs/about-claude/claude-code)** and the **[Firecrawl](https://www.firecrawl.dev/)** Model Context Protocol (MCP) Connector. This system automates the traditional, week-long manual GTM (Go-To-Market) pipeline—comprising list building, manual research, and copywriting—into a repeatable, self-correcting 20-minute automated run.

The agent autonomously scrapes highly targeted prospect lists, enriches them with deep contextual company data, self-heals by routing around LinkedIn blockages to corporate websites, writes highly personalized cold outreach emails, and compiles the final output into a ready-to-use CSV.

---

## Table of Contents

1. [Workflow Pipeline Overview](#workflow-pipeline-overview)
2. [Prerequisites & MCP Integration](#prerequisites--mcp-integration)
3. [Step-by-Step Execution Guide](#step-by-step-execution-guide)
4. [Creating a Repeatable Claude Skill](#creating-a-repeatable-claude-skill)
5. [Critical Lessons & Human-in-the-Loop Guardrails](#critical-lessons--human-in-the-loop-guardrails)

---

## Workflow Pipeline Overview

This system automates three critical phases of the B2B outreach funnel into a unified agentic sequence:

```
[Target Criteria]
       │
       ▼
┌────────────────────────────────────────────────────────┐
│ Stage 1: Lead Scraping & List Building                 │ (Scrapes directories & LinkedIn)
└──────────────────────────┬─────────────────────────────┘
                           │ Outputs: leads.csv
                           ▼
┌────────────────────────────────────────────────────────┐
│ Stage 2: Autonomous Information Enrichment             │ (Visits sites, handles blocks)
└──────────────────────────┬─────────────────────────────┘
                           │ Outputs: leads_enrich.csv
                           ▼
┌────────────────────────────────────────────────────────┐
│ Stage 3: Context-Aware Copywriting                      │ (Generates custom email hooks)
└──────────────────────────┬─────────────────────────────┘
                           │ Outputs: leads_final.csv
                           ▼
                   [Human Review]
```

1. **Lead Scraping & List Building**: The agent receives natural language criteria (e.g., location, company size, target job titles) and searches LinkedIn and business directories to compile contact cards.
2. **Autonomous Information Enrichment**: The agent attempts to scrape prospect activity or corporate updates, gracefully switching targets if a platform blocks extraction.
3. **Context-Aware Outreach Copywriting**: The agent synthesizes the gathered prospect context to draft tailored, hyper-personalized emails featuring custom hooks and low-friction call-to-actions (CTAs).

---

## Prerequisites & MCP Integration

### Prerequisites

- **[Claude Code](https://docs.anthropic.com/en/docs/about-claude/claude-code)**: Accessible via your local computer terminal, or via the **[Claude Desktop App](https://claude.ai/download)**.
- **[Firecrawl](https://www.firecrawl.dev/)**: A web scraper and crawler that strips away messy HTML, optimizing data extraction and cutting token consumption costs. Sign up at the **[Firecrawl Website](https://www.firecrawl.dev/)**. The free plan offers 1,000 credits/month, with paid tiers starting at $16/month (annual plan).
- **[Firecrawl MCP Server Documentation](https://docs.firecrawl.dev/features/mcp)**: Official guide on running Firecrawl MCP on Claude.

### Connecting Firecrawl as a Custom MCP Server

Because Firecrawl is not included in Claude's default browse connectors, you must register it as a custom remote Model Context Protocol (MCP) server:

1. Open your Claude Code/Desktop client and navigate to **Add Custom Connector**.
2. Name the connector: `firecrawl`.
3. Provide the remote MCP server URL. Locate the hosted URL pattern in the **[Firecrawl MCP Documentation](https://docs.firecrawl.dev/features/mcp)**:
   `https://mcp.firecrawl.dev/?apiKey=YOUR_FIRECRAWL_API_KEY` (ensure you replace `YOUR_FIRECRAWL_API_KEY` with your actual API key from the **[Firecrawl Dashboard](https://www.firecrawl.dev/app/dashboard)**).
4. Submit to establish the connection.

Verify the connection within Claude Code by executing:

```bash
please test if you are connected to fire crawl
```

Confirm that Claude responds with a successful connection status before initiating the workflow.

---

## Step-by-Step Execution Guide

To optimize token usage and ensure output accuracy, it is highly recommended to run this pipeline in incremental batches (e.g., testing first on 5 prospects before scaling to 50+).

### Stage 1: Scrape & Build the Core List

Instruct the agent to seek out prospects matching your exact ICP (Ideal Customer Profile) and format the data into a structured CSV:

> **Prompt:**
> _"Use firecrawl to find 20 HR directors at companies with 50 to 500 employees in San Jose, CA or Santa Clara, CA or Palo Alto, CA or Mountain View, CA. Search LinkedIn and business directories. For each lead, collect full name, job title, company name, company website, and LinkedIn URL if available. Save this all to a CSV file called leads.csv."_

- **Agent Behavior**: Claude runs multiple targeted searches in parallel across public business indices and LinkedIn, compiles the fields, and exports them directly into `leads.csv`.

### Stage 2: Deep Enrichment & Self-Healing Fallback

Using the initial list, instruct the agent to research each prospect to find unique, leverageable angles for personalization.

> **Prompt:**
> _"Read leads.csv. For each lead that has a LinkedIn URL, visit their profile using firecrawl and find the most recent post or activity. Add two columns to the CSV: 'recent_topic' (a 5 to 10-word summary of what they've been posting about) and 'enrichment_note' (one sentence I could reference in an outreach email). Save this as leads_enrich.csv. If LinkedIn is blocked, switch to using company websites instead.Do this for the first 5 leads"_

- **Self-Healing Mechanics**: LinkedIn rigorously blocks automated scrapers. When Firecrawl encounters a block, the agent automatically adapts by pivoting to the prospect's company website. It scrapes the site for recent press releases, acquisitions, product announcements, or anniversaries, writing alternative corporate enrichment notes.

### Stage 3: Draft Personalized Outreach

Combine the contact info and enrichment notes to generate tailored cold outreach campaigns:

> **Prompt:**
> _"Read leads_enrich.csv. For each lead, write a short cold outreach email: 1) Open with a specific reference to their recent topic or enrichment note, if any. 2) Introduce me as [Your Name], who helps midsize companies streamline HR operations using AI tools. 3) End with one low-commitment CTA, such as a quick reply to learn more. Add the email as a new column called 'draft_email' and save it as leads_final.csv."_

- **Outreach Personalization**: The resulting `leads_final.csv` will contain the complete prospect profile, the extracted research context, and a customized cold email ready for review.

---

## Creating a Repeatable Claude Skill

To transform this multi-stage workflow from a one-off run into a highly reusable corporate utility, you can package the entire logic into a **Claude Skill**.

Instruct Claude Code:

> **Prompt:**
> _"I would like to use all of this again. Please save this as a skill file for future reference."_

Claude packages the process and saves it as a reusable skill (e.g., `lead gen HR skill`) in your Claude skills library. In future sessions, you can invoke the entire automated pipeline with a single simple natural language command:

```bash
find 20 HR directors at 50 to 500 employee companies in San Jose
```

The agent will automatically trigger the 3-stage scrape-enrich-draft pipeline for Dallas without requiring you to re-enter individual prompts.

---

## Critical Lessons & Human-in-the-Loop Guardrails

While this AI-agentic pipeline dramatically speeds up B2B prospecting, humans must remain "in-the-loop" to guarantee high-quality outreach:

1. **Verify Outreach Relevance**: AI scrapers can occasionally make mistakes or scrape incomplete profiles. Always review the generated leads and notes to make sure they match your target buyer persona.
2. **Bad Personalization is Worse Than No Personalization**: If an enrichment note incorrectly interprets a prospect's role or references an incorrect company announcement, it undermines your credibility. If a profile cannot be cleanly enriched, fall back to a clean, well-written generic outreach email rather than a flawed personalization attempt.
3. **Respect Scraping Policies**: Always check and comply with the target platform's Terms of Service and robot.txt directives.
