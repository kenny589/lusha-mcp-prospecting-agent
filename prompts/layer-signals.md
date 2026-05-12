# Prompt 2: Layer Buying Signals

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

Example request body:
{
  "companyIds": [12345, 67890],
  "signals": ["surgeInHiring", "headcountIncrease1m", "headcountIncrease3m", "financialEventsNews", "productActivityNews", "peopleNews"]
}

This costs credits — tell me how many companies you're checking before running it.

For each company with at least one signal, show me:
- Company name and domain
- Signal type
- Signal date
- Key data point (e.g., "546 new job postings, 2.5x historical average" or "headcount grew 12% in 1 month")
```

## Customization Tips

- Use `"allSignals"` instead of specific ones to get everything (costs more credits)
- Add `"websiteTrafficIncrease"` if you sell marketing tools
- Add `"itSpendIncrease"` if you sell to IT buyers
- Add `"corporateStrategyNews"` if you target companies opening new locations or restructuring
- Add `"surgeInHiringByDepartment"` to see which departments are growing
- Add `"riskNews"` if you sell compliance or security tools
- The API looks back 6 months by default
