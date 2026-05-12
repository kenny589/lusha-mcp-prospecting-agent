# Claude Code x Lusha: AI Prospecting Agent

Build an AI prospecting agent in Claude Code using Lusha's MCP server, Prospecting API, and Signals API.

Open Claude Code. Describe your ICP. Get verified contacts, company data, and live buying signals — in minutes.

## What Claude Code Does With Lusha

You give Claude Code a natural language prompt. It handles the rest:

1. Searches for matching companies and contacts via Lusha's Prospecting API
2. Layers buying signals (hiring surges, headcount growth, funding, news) from the Signals API
3. Enriches top contacts with verified emails and direct dials via MCP
4. Ranks every account as HOT / WARM / WATCH
5. Returns an action-ready prospecting queue

No spreadsheets. No tab switching. No manual enrichment. Just Claude Code + Lusha.

## Setup in Claude Code (5 minutes)

### 1. Get a Lusha API Key

- [Create a free Lusha account and get your API key here](https://partnerstack.lusha.com/Kennydamian)
- Go to **Settings > API**
- Generate your production API key

### 2. Add the Hosted MCP Server to Claude Code

Lusha has a hosted MCP server with 9 tools — including full prospecting search, contact enrichment, and company enrichment. No npm install needed.

Add this to your Claude Code MCP settings (`.claude/mcp.json` or via the Claude Code settings UI):

```json
{
  "mcpServers": {
    "lusha": {
      "type": "url",
      "url": "https://mcp.lusha.com/mcp",
      "headers": {
        "x-api-key": "YOUR_LUSHA_API_KEY"
      }
    }
  }
}
```

### 3. Set Your API Key for REST Calls

The Prospecting API and Signals API are called via REST (not MCP). Set your API key as an environment variable:

```bash
export LUSHA_API_KEY="your_api_key_here"
```

Add this to your shell profile (`.zshrc` or `.bashrc`) so it persists.

### 4. Add the CLAUDE.md System Prompt

Copy the `CLAUDE.md` file from this repo into your project root. This gives Claude Code everything it needs — API schemas, signal types, filter options, enrichment flow — so Claude Code can execute the entire prospecting workflow from a single natural language prompt.

### 5. Verify the Connection

Open Claude Code and type:

```
What Lusha tools do you have access to?
```

Claude should list these 9 tools:

| Tool | What It Does | Credits |
|------|-------------|---------|
| `prospecting_search_guide` | Workflow guidance for prospecting | Free |
| `prospecting_company_filters` | Get available company filter options | Free |
| `prospecting_company_search` | Search companies by ICP filters | Free |
| `prospecting_company_enrich` | Enrich companies from search results | 1+ per company |
| `prospecting_contact_filters` | Get available contact filter options | Free |
| `prospecting_contact_search` | Search contacts by title, seniority, etc. | Free |
| `prospecting_contact_enrich` | Reveal verified emails and phones | 1+ per contact |
| `contacts_search` | Enrich contacts by LinkedIn URL or name | 1 per contact |
| `companies_search` | Enrich companies by domain or name | 1 per company |

## How the Agent Works

```
                        Claude Code
 ┌──────────────────────────────────────────────────────┐
 │                                                      │
 │  1. ICP Prompt                                       │
 │     ↓                                                │
 │  2. Company Search ──→ Lusha Prospecting API (REST)  │
 │     ↓                   POST /prospecting/company/   │
 │  3. Contact Search ──→  POST /prospecting/contact/   │
 │     ↓                                                │
 │  4. Buying Signals ──→ Lusha Signals API (REST)      │
 │     ↓                   POST /api/signals/companies  │
 │  5. Enrich Top     ──→ Lusha MCP Server              │
 │     Contacts            prospecting_contact_enrich   │
 │     ↓                                                │
 │  6. Rank & Output                                    │
 │     HOT / WARM / WATCH                               │
 │                                                      │
 └──────────────────────────────────────────────────────┘
```

**Steps 2-3 (Search)** are free — no credits consumed. You only pay when you enrich contacts or pull signals.

## API Reference (What Claude Calls Under the Hood)

### Company Prospecting Search

```
POST https://api.lusha.com/prospecting/company/search
Header: api_key: YOUR_KEY
```

```json
{
  "filters": {
    "companies": {
      "include": {
        "sizes": [{"min": 50, "max": 500}],
        "mainIndustriesIds": [6],
        "locations": [{"country": "United States"}],
        "technologies": ["HubSpot", "Salesforce"],
        "searchText": "B2B SaaS"
      }
    }
  },
  "pages": {"page": 0, "size": 25}
}
```

**Filter options:**
- `sizes` — Array of `{min, max}` employee count ranges
- `mainIndustriesIds` — Industry IDs (get from `/prospecting/company/filters/industries`)
- `locations` — Array of `{country}` and/or `{state}` objects
- `technologies` — Tech stack filter (e.g. "HubSpot", "Salesforce", "Marketo")
- `searchText` — Free-text company name or keyword search
- `names` — Exact company names
- `domains` — Exact domains
- `revenues` — Revenue ranges
- `sicCodes`, `naicsCodes` — Industry codes

**Pagination:** `page` (0-1000), `size` (10-50, minimum 10)

**Cost:** Free. Search does not consume credits.

### Contact Prospecting Search

```
POST https://api.lusha.com/prospecting/contact/search
Header: api_key: YOUR_KEY
```

```json
{
  "filters": {
    "contacts": {
      "include": {
        "jobTitles": ["VP Sales", "Head of Revenue", "CRO"],
        "seniority": [7, 8, 9],
        "departments": ["sales"],
        "locations": [{"country": "United States"}]
      }
    },
    "companies": {
      "include": {
        "sizes": [{"min": 50, "max": 500}],
        "names": ["Acme Inc"]
      }
    }
  },
  "pages": {"page": 0, "size": 25}
}
```

**Contact filter options:**
- `jobTitles` — Job title keywords
- `seniority` — Seniority level IDs: 10 (Founder), 9 (C-Suite), 8 (VP), 7 (Director), 6 (Manager), 5 (Senior), 4 (Entry), 3 (Training)
- `departments` — Department names (sales, marketing, engineering, etc.)
- `locations` — Contact location
- `linkedinUrls` — Direct LinkedIn profile URLs
- `searchText` — Free-text search
- `signal` — Filter by buying signal (`names` + `startDate`)

**Cost:** Free. Search does not consume credits.

### Contact Enrichment

```
POST https://api.lusha.com/prospecting/contact/enrich
Header: api_key: YOUR_KEY
```

```json
{
  "requestId": "uuid-from-search-response",
  "contactIds": ["contact-uuid-1", "contact-uuid-2"]
}
```

**Returns:** Verified work emails (with confidence score A+/A/B), direct phone numbers, mobile numbers, LinkedIn URL, department, seniority, full company data.

**Cost:** Credits per contact (varies based on data points revealed — typically 1-6 credits).

### Company Signals

```
POST https://api.lusha.com/api/signals/companies
Header: api_key: YOUR_KEY
```

```json
{
  "companyIds": [12790225, 83087822],
  "signals": ["surgeInHiring", "headcountIncrease1m", "financialEventsNews", "productActivityNews"]
}
```

**Available company signals:**

| Signal | What It Detects |
|--------|----------------|
| `surgeInHiring` | Unusual spike in job postings |
| `surgeInHiringByDepartment` | Hiring surge in specific departments |
| `surgeInHiringByLocation` | Hiring surge in specific locations |
| `headcountIncrease1m` / `3m` / `6m` / `12m` | Employee count growth over time |
| `headcountDecrease1m` / `3m` / `6m` / `12m` | Employee count decline over time |
| `websiteTrafficIncrease` | Website traffic growth |
| `websiteTrafficDecrease` | Website traffic decline |
| `itSpendIncrease` | IT budget expansion |
| `itSpendDecrease` | IT budget contraction |
| `financialEventsNews` | Funding rounds, IPO, investment activity |
| `commercialActivityNews` | Partnerships, deals, commercial moves |
| `corporateStrategyNews` | Restructuring, new locations, strategy shifts |
| `productActivityNews` | Product launches, features, integrations |
| `peopleNews` | Leadership changes, key hires |
| `marketIntelligenceNews` | Market positioning shifts |
| `riskNews` | Legal, security, or compliance flags |
| `allSignals` | Returns all signal types |

**Cost:** Credits charged per company (varies — our test returned 7 credits for 1 company with 4 signal types). Default lookback: 6 months.

### Contact Signals

```
POST https://api.lusha.com/api/signals/contacts
Header: api_key: YOUR_KEY
```

**Available contact signals:** `promotion`, `companyChange`

### Person Enrichment (v2)

```
POST https://api.lusha.com/v2/person
Header: api_key: YOUR_KEY
```

```json
{
  "contacts": [
    {
      "contactId": "my-ref-1",
      "linkedinUrl": "https://www.linkedin.com/in/someone"
    }
  ]
}
```

**Alternative inputs:** `firstName` + `lastName` + `company`, or `email`.

**Cost:** 1 credit per contact.

## Prompt Templates for Claude Code

Copy these prompts directly into Claude Code to run the full prospecting workflow.

### Prompt 1: Search Your ICP

```
I'm building a prospect list. Here's my ICP:

- Target titles: VP Sales, Head of Revenue, CRO
- Company size: 50 to 500 employees
- Industry: B2B SaaS
- Region: United States
- Exclude: Agencies, consulting firms, companies under $2M ARR

Search Lusha's Prospecting API for companies matching these filters.
Then search for contacts at those companies with my target titles.
Return up to 25 results.
```

### Prompt 2: Layer Buying Signals

```
Now check which of these companies have active buying signals.

Pull these signals for every company from the search results:
- Hiring surges (surgeInHiring)
- Headcount growth (headcountIncrease1m, headcountIncrease3m)
- Financial events / funding (financialEventsNews)
- Product launches (productActivityNews)
- Leadership changes (peopleNews)

Use Lusha's Signals API. Flag every company with at least one active signal.
```

### Prompt 3: Enrich the Decision-Makers

```
For each company with an active signal, enrich the best contact.

Priority titles: VP Sales > Head of Revenue > CRO > Director of Sales

Use Lusha MCP to pull:
- Verified work email
- Direct phone number
- Current title and seniority
- LinkedIn profile URL

Skip any contact without a verified email.
```

### Prompt 4: Rank and Output

```
Rank every account using this system:

HOT = Strong ICP fit + active buying signal in the last 30 days
WARM = ICP fit, signal is older than 30 days or weak
WATCH = Partial ICP fit, no signal, worth monitoring

For each account, return:
- Company name and domain
- Best contact (name, title, email, phone)
- Signal type and date
- Rank (HOT / WARM / WATCH)
- One-line outreach angle tied to the signal

Format as a table. Sort HOT first, then WARM, then WATCH.
```

### Single-Prompt Version

You can also run the entire workflow in one shot:

```
Build me a prospecting list using Lusha.

ICP:
- Titles: VP Sales, Head of Revenue, CRO, Director of Sales
- Company size: 50-500 employees
- Industry: B2B SaaS
- Region: United States

For each company:
1. Search for matching companies via Lusha's Prospecting API
2. Check for buying signals (hiring surges, headcount growth, funding, product launches)
3. Find the best contact at each company and enrich with verified email + phone
4. Rank as HOT (signal in last 30 days), WARM (older signal), or WATCH (no signal)

Return a ranked table with: company, domain, contact name, title, email, phone, signal, rank, and a one-line outreach angle.
```

## Ranking System

| Rank | Criteria | Action |
|------|----------|--------|
| **HOT** | Strong ICP fit + active signal in last 30 days | Outreach immediately |
| **WARM** | ICP fit, signal older than 30 days or weak | Add to nurture sequence |
| **WATCH** | Partial ICP fit, no signal | Monitor for future signals |

## Sample Output

| Company | Contact | Title | Email | Signal | Rank | Outreach Angle |
|---------|---------|-------|-------|--------|------|----------------|
| Acme Inc | Sarah Chen | VP Sales | s.chen@acme.com | Hiring Surge (546 new postings) | HOT | "Noticed Acme is on a hiring tear — usually means outbound infrastructure needs to scale fast." |
| Bolt.io | James Park | Head of RevOps | j.park@bolt.io | Headcount +12% (1mo) | HOT | "Bolt grew 12% last month. When teams scale that fast, outbound usually breaks first." |
| Kova | Maria Lopez | Dir. Marketing | m.lopez@kova.com | Product Launch | WARM | "Saw Kova just shipped a new integration — new products need new pipeline." |
| Nova Labs | David Kim | CRO | d.kim@novalabs.com | None | WATCH | — |

## Credits and Pricing

| Action | Cost |
|--------|------|
| Company search | Free |
| Contact search | Free |
| Filter options | Free |
| Contact enrichment | 1-6 credits per contact |
| Company enrichment | 1+ credits per company |
| Company signals | Varies (1-10+ credits per batch) |
| Contact signals | 1 credit per contact |

**Rate limit:** 25 requests/second per endpoint.

## Repo Structure

```
lusha-mcp-prospecting-agent/
├── README.md          # This file — setup, API reference, prompts
├── CLAUDE.md          # System prompt for Claude Code (drop into your project)
├── prompts/
│   ├── search-icp.md          # Prompt 1: Search by ICP
│   ├── layer-signals.md       # Prompt 2: Add buying signals
│   ├── enrich-contacts.md     # Prompt 3: Enrich decision-makers
│   ├── rank-and-output.md     # Prompt 4: Rank and format
│   └── single-prompt.md       # All-in-one version
└── examples/
    └── sample-output.md       # Example ranked output table
```

## Resources

- [Lusha MCP Docs](https://docs.lusha.com/mcp-docs)
- [Lusha API Docs](https://docs.lusha.com)
- [Lusha OpenAPI Spec](https://docs.lusha.com/apis/openapi)
- [Get a Free API Key](https://partnerstack.lusha.com/Kennydamian)
- [Lusha npm MCP Package](https://github.com/lusha-oss/lusha-public-api-mcp) (enrichment-only, 2 tools)

---

Built by [Kenny Damian](https://linkedin.com/in/kennydamian) @ [ColdIQ](https://coldiq.com)
