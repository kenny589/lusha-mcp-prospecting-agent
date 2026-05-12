# Single-Prompt Version (All-in-One)

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

This single prompt runs the entire 4-step workflow in one shot. Claude Code will execute all the API calls sequentially and return the final ranked table.
