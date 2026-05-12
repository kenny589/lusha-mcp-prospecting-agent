# Prompt 3: Enrich the Decision-Makers

```
For each company with an active buying signal, enrich the best contact.

Priority titles (in order): VP Sales > Head of Revenue > CRO > Director of Sales

Use the Lusha MCP tool "prospecting_contact_enrich" if available.
Or use the REST endpoint: POST https://api.lusha.com/prospecting/contact/enrich
With the api_key header for authentication.

You MUST pass the requestId from the contact search response — it won't work without it.
Pass the contactIds (UUIDs) of the contacts you want to enrich.

Example request body:
{
  "requestId": "uuid-from-contact-search",
  "contactIds": ["contact-uuid-1", "contact-uuid-2"]
}

This costs 1-6 credits per contact depending on data revealed. Tell me how many contacts and estimated credit cost before running it.

For each enriched contact, show me:
- Full name
- Current title
- Company name and domain
- Verified work email (note the confidence score: A+, A, or B)
- Direct phone number (if available)
- Mobile phone number (if available)
- LinkedIn profile URL

Skip any contact that comes back without a verified email.
```

## Customization Tips

- Change the title priority order to match your buyer persona
- Add "Include all email addresses, not just the primary" for multi-threading
- Add "Flag contacts with A+ email confidence separately" for deliverability prioritization
- Remove "Skip contacts without verified email" if you want the full list
- Add "Also enrich the company" to get firmographic data (revenue, tech stack, funding)
