# Sample Input

Paste each block below into the matching field of the **Founder's Weekly Operating System** form when you run the workflow. This data is designed to show every part of the system — including a recurring bottleneck the AI should detect.

---

### Meeting Notes
```
Mon standup: Engineering is blocked on the payments migration — Stripe webhooks failing intermittently.
Sales call with Acme Corp (potential 40k/yr deal) — they want SSO before signing.
Investor update is due Friday; deck not started yet.
Decided to pause the mobile app work until the web onboarding is fixed.
Two engineers out next week, so capacity is tight.
```

### Customer Feedback
```
"I couldn't figure out how to set up my account — gave up after 10 minutes."
"Onboarding is way too confusing, took me forever to get started."
"Signup worked but I had no idea what to do next, the first-run experience is rough."
"Love the product once you're in, but getting started is painful."
"Support replied fast, that was great."
"The new dashboard is clean."
"Setup instructions were unclear, almost churned during the trial."
"Pricing page is confusing too, wasn't sure which plan to pick."
```

### Slack & Emails
```
#eng: webhook retries still failing in prod, 3rd day in a row. Need a fix.
Email from Acme procurement: "Can you confirm SSO support timeline?"
#support: 4 new tickets today all about onboarding/setup confusion.
Email from advisor: "Did you send the investor update yet?"
#general: reminder — board call moved to next Tuesday.
```

### Current Task List
```
- Fix Stripe webhook failures
- Build SSO for enterprise
- Write + send investor update (due Fri)
- Rewrite onboarding/setup flow
- Hire a second support person
- Refresh pricing page copy
- Reply to Acme procurement
```

---

## What a good run should produce

- **Agent 1** extracts ~7 tasks, several onboarding/setup feedback items, and the webhook + SSO issues.
- **Agent 2** rates the webhook fix and SSO high on urgency/revenue (Acme deal + prod outage).
- **Score & Rank** (deterministic) puts the Stripe webhook fix and SSO near the top, bucketed *Critical*.
- **Agent 3 + Flag Bottlenecks** detects **onboarding/setup confusion** appearing ~5–6 times → `systemic: true`.
- **Agent 4 (CEO)** returns top-3 priorities, names onboarding (or the prod webhook outage) as the biggest risk, lists quick wins (reply to Acme, send investor update) and delegate items.
- The **approval form** shows the plan; choosing **Approve** finalises it. See `sample_output.json` for a representative result.
