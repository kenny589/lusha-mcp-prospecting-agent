# Prompt 1: Search Your ICP

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

## Customization Tips

- Swap the titles for your buyer persona (e.g., "VP Marketing, Head of Demand Gen, CMO")
- Change seniority IDs: 10=Founder, 9=C-Suite, 8=VP, 7=Director, 6=Manager, 5=Senior
- Change the company size range to match your segment (startups: 1-50, mid-market: 51-500, enterprise: 500+)
- Add technology filters: add `"technologies": ["HubSpot"]` to the company filters
- Add location specificity: use `{"state": "California"}` instead of country-level
- Use `searchText` for keyword matching (e.g., "AI", "fintech", "healthcare")
