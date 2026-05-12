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

Copy these prompts directly into Claude Code. Each one tells Claude Code exactly what API to call, what headers to use, and what format to return. See the `prompts/` folder for expanded versions with customization tips.

### Prompt 1: Search Your ICP

```
I'm building a prospect list using Lusha. Here's my ICP:

- Target titles: VP Sales, Head of Revenue, CRO
- Company size: 50 to 500 employees
- Industry: B2B SaaS
- Region: United States
- Exclude: Agencies, consulting firms, companies under $2M ARR

Step 1: Search for companies matching my ICP.
Use the Lusha Prospecting API: POST https://api.lusha.com/prospecting/company/search
Use the api_key header for authentication with my LUSHA_API_KEY env variable.
Set sizes as [{"min": 50, "max": 500}] and locations as [{"country": "United States"}].
Use searchText for industry keywords if needed. Return 25 results.

Step 2: For each company returned, search for contacts matching my target titles.
Use: POST https://api.lusha.com/prospecting/contact/search
Filter by jobTitles: ["VP Sales", "Head of Revenue", "CRO"] and seniority: [7, 8, 9] (Director, VP, C-Suite).
Pass the company names or domains from Step 1 into the companies filter.

Both searches are FREE — no credits consumed.

Show me the results as a table with: company name, domain, contact name, title, and whether they have email/phone available.
```

### Prompt 2: Layer Buying Signals

```
Now check which of these companies have active buying signals.

Use the Lusha Signals API: POST https://api.lusha.com/api/signals/companies
Use the api_key header for authentication.

Pass the companyIds (integers) from the search results as an array.
Max 100 company IDs per request.

Pull these specific signals:
- "surgeInHiring" — unusual spike in job postings
- "headcountIncrease1m" — employee growth in last month
- "headcountIncrease3m" — employee growth in last 3 months
- "financialEventsNews" — funding rounds, IPO, investment
- "productActivityNews" — product launches, new features
- "peopleNews" — leadership changes, key hires

This costs credits — tell me how many companies you're checking before running it.

For each company with at least one signal, show me: company name, domain, signal type, signal date, and the key data point.
```

### Prompt 3: Enrich the Decision-Makers

```
For each company with an active buying signal, enrich the best contact.

Priority titles (in order): VP Sales > Head of Revenue > CRO > Director of Sales

Use the Lusha MCP tool "prospecting_contact_enrich" if available.
Or use the REST endpoint: POST https://api.lusha.com/prospecting/contact/enrich
With the api_key header for authentication.

You MUST pass the requestId from the contact search response + the contactIds (UUIDs).

This costs 1-6 credits per contact. Tell me how many contacts and estimated credit cost before running it.

For each enriched contact, show me: full name, title, company, verified work email (with confidence score), direct phone, mobile phone, LinkedIn URL.
Skip any contact without a verified email.
```

### Prompt 4: Rank and Output

```
Rank every account:

HOT = ICP fit + at least one buying signal in the last 30 days
WARM = ICP fit + signal older than 30 days or weak signal
WATCH = Partial ICP fit, no active signal

Return a markdown table with: company, domain, contact name, title, email, phone, signal type, signal date, rank, and a one-line outreach angle that references the specific signal data (e.g., "546 new job postings" not just "hiring surge").

Sort HOT first, then WARM, then WATCH.
Show a summary: count per tier, total credits used, suggested next step per tier.
Export as CSV.
```

### Single-Prompt Version (All-in-One)

Run the entire workflow in one shot — paste this into Claude Code:

```
Build me a prospecting list using Lusha's APIs. My LUSHA_API_KEY is set as an environment variable.

ICP:
- Titles: VP Sales, Head of Revenue, CRO, Director of Sales
- Company size: 50-500 employees
- Industry: B2B SaaS
- Region: United States

Run these steps in order:

1. SEARCH COMPANIES (free — no credits)
   POST https://api.lusha.com/prospecting/company/search
   Header: api_key from my LUSHA_API_KEY env variable
   Use sizes as [{"min": 50, "max": 500}], locations as [{"country": "United States"}].
   Use searchText for industry keywords. Return 25 results.

2. SEARCH CONTACTS at those companies (free — no credits)
   POST https://api.lusha.com/prospecting/contact/search
   Filter by jobTitles: ["VP Sales", "Head of Revenue", "CRO", "Director of Sales"]
   Filter by seniority: [7, 8, 9] (Director, VP, C-Suite)
   Pass company names from Step 1 into the companies filter.

3. CHECK BUYING SIGNALS (costs credits — tell me before running)
   POST https://api.lusha.com/api/signals/companies
   Pass companyIds (integers) from search results.
   Check signals: ["surgeInHiring", "headcountIncrease1m", "headcountIncrease3m", "financialEventsNews", "productActivityNews", "peopleNews"]

4. ENRICH TOP CONTACTS (costs credits — tell me before running)
   POST https://api.lusha.com/prospecting/contact/enrich
   Pass the requestId from the contact search response + contactIds of best contacts.
   Only enrich contacts at companies with active signals.

5. RANK AND OUTPUT
   HOT = ICP fit + signal in last 30 days
   WARM = ICP fit + older/weak signal
   WATCH = partial fit, no signal

   Return a markdown table: company, domain, contact name, title, email, phone, signal type, signal date, rank, one-line outreach angle (must reference specific signal data).
   Export as CSV. Show total credits used.
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

Built by [Kenny Damian](https://www.linkedin.com/in/kenny-damian-90aba221a/) @ [ColdIQ](https://coldiq.com)

[Get a Free Lusha API Key](https://partnerstack.lusha.com/Kennydamian)
