# Prompt 4: Rank and Output

```
Rank every account using this system:

HOT = Strong ICP fit + at least one buying signal dated within the last 30 days
WARM = ICP fit + signal older than 30 days, or a weak signal (e.g., small headcount change)
WATCH = Partial ICP fit, no active signal, worth monitoring

For each account, return a table with these columns:
- Company name
- Domain
- Contact name
- Title
- Verified email
- Phone number
- Signal type (e.g., "Hiring Surge", "Headcount +12%", "Series B Funding")
- Signal date
- Rank (HOT / WARM / WATCH)
- One-line outreach angle — this MUST reference the specific signal data (e.g., "546 new job postings" not just "hiring surge")

Sort the table: HOT first, then WARM, then WATCH.

At the bottom, add a summary:
- Count of HOT / WARM / WATCH accounts
- Total credits used across all API calls
- Suggested next step for each tier (e.g., "Outreach immediately", "Add to nurture", "Monitor")

Also export the full table as a CSV file I can download.
```

## Customization Tips

- Change the ranking thresholds (e.g., "HOT = signal in last 14 days" for more urgency)
- Add "Write a 3-sentence cold email for each HOT prospect" for immediate outreach
- Add "Include LinkedIn profile URL" for social selling workflows
- Add "Group by signal type" to see all hiring surge companies together
- Add "Score each account 1-100" for more granular prioritization
