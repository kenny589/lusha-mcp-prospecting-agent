# Single-Prompt Version (All-in-One)

Paste this directly into Claude Code. It runs the entire 4-step workflow in one shot.

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
   Only enrich contacts at companies with active signals. Skip the rest.

5. RANK AND OUTPUT
   HOT = ICP fit + signal in last 30 days → outreach immediately
   WARM = ICP fit + older/weak signal → nurture sequence
   WATCH = partial fit, no signal → monitor

   Return a markdown table: company, domain, contact name, title, email, phone, signal type, signal date, rank, one-line outreach angle (must reference specific signal data).

   Export as CSV. Show total credits used.
```

## How It Works

Claude Code reads the CLAUDE.md system prompt (which has all the API schemas, gotchas, and authentication details), then executes each step sequentially. The searches are free, so Claude will run those first, then pause to confirm credit costs before enriching or pulling signals.
