# Prompt 2: Layer Buying Signals

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

## Customization Tips

- Use `allSignals` instead of specific ones to get everything (costs more credits)
- Add `websiteTrafficIncrease` if you sell marketing tools
- Add `itSpendIncrease` if you sell to IT buyers
- Add `corporateStrategyNews` if you target companies in expansion mode
