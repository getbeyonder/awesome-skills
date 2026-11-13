# Phase 2: Enrich

Find a verified work email for each person in the list. Only Dealmaker is used; it asks the platform, which asks
the email providers. Skip this phase when the package chose LinkedIn only. Held people can be enriched: admission is
not needed for a lookup.

## Showing a price

Use this for every paid quote in the flow.
- **The quote has `maxCredits`:** "up to <maxCredits> credits (~$<maxPriceUsd>)". Per item, if given:
  `perLookupMaxCredits` / `perPersonMaxCredits`.
- **No credit fields, but a cost field** (`maxUsd`, `perPersonMaxUsd`, `capUsd`; `prospects.research` often answers
  only these): that is the platform's cost, never shown. Convert `perPersonMaxUsd` for the price per person and
  `maxUsd` for the total; `capUsd` is the tool's hard cap per person, not the quote (use it only as an upper
  bound). Credits = cost in USD × 100, rounded up (1 credit per cent of cost); no `maxUsd`: total = credits per
  item × the number of items; dollars in brackets = credits × the price per credit, which is a `people.search`
  quote's `maxPriceUsd` ÷ `maxCredits` (else about $0.015). Example:
  $0.038 cost → 4 credits (~$0.06). Never divide a cost by the price per credit. Say "about" and that the price is
  converted (the app gave no credit price).
- **No price at all** (an older email lookup answers a `quote` with `total: 0`, `available: 0`): that 0 is not the
  price; never show it. Say the app gives no price for this, and use the typical ceiling of about 29 credits
  (~$0.43) per email found as an estimate, clearly marked as one.
- Credits are what the company's wallet is charged; the dollars in brackets are what those credits sell for.

## The budget and the ledger

Use this for every paid call in the flow (`people.search`, `prospects.enrich`, `prospects.research`).

**The budget** is the hard ceiling in credits from the approved package. **The ledger** is your running count in
this chat of what may have been spent: for each real paid call, its quoted `maxCredits` (the most it can cost).
Once a result shows fewer billable items, lower that entry: a search bills only people returned (`found` × the
per-person ceiling); an email lookup bills only emails found (found × the per-email ceiling). Research stays at its
quote. Always say "up to", since the app does not report the final charge.

**Go without asking** when all three hold:
1. **Budget:** the quote's `maxCredits` ≤ budget − ledger.
2. **Scope:** the same filters, count and list as the package (for Enrich: people in the run's list; for Research:
   the run's list, `days` as in the package).
3. **Estimate:** the price per unit (per person searched, per email, per person researched) is at most 25% above
   the package's estimate for that unit.

**Stop and ask** when any of them fails, and on `outcome_unknown`, `payment_required`, `no_credits` or any refusal.
Say what happened, the numbers, and a smaller proposal that fits ("The lookups for 21 people can cost up to 609
credits, but 480 are left. I can look up the first 16 now, or you raise the budget. Which?"). A raise of the budget
is the person's explicit new number; the ledger keeps counting.

**After every paid step** show one line: "Spent so far: up to <ledger> of <budget> credits (~$<x>); left: <budget
− ledger>."

## 1. Is email lookup on?

`app_read(warmleads, enrichment.status)` again (it can change after Setup). `kinds` has `email`: go on.
`status: "not_configured"`: say so, skip to Research; the campaign runs on LinkedIn only (say it in the progress
report: it changes the package's channels). Never use `kind: "phone"` (phones are out of scope; a phone lookup is a
person's step in the app).

No connected mailbox yet: go on (the package planned email), but say in the report that emails found cannot be used
until the mailbox is connected, before Launch.

## 2. Who needs a lookup

`prospects.search` with `filters.listId` (follow `cursor`). Everyone without an email whose `state` is not
`suppressed`. That is everyone in the list (the package's default), minus anyone the person asked to remove.

## 3. Dry runs and the quote

One `prospects.enrich` call handles one person: `{ prospectId, kind: "email" }`.
- Dry run per person. The answer: `quote` (with `free: 1` when the email is already known: free, nothing new is
  looked up; `payable`; credit fields when present) or `status: "unavailable"` with a `reason` (no provider, no
  price): leave that person out and say why.
- Add up the payable people and their credits. Check budget, scope and estimate (above). All fit: go on.
- Over what is left: stop and ask, with the part that fits as the proposal.

## 4. The real calls

The same `args` per person, without `dry_run`, each with its own fresh `retryKey` if the schema lists it. The answer
has `jobIds`: the lookup is queued, not done, and the answer has no email yet. `ENRICHMENT_ALREADY_PENDING`: a lookup
for this person is already running or ended unclear; do not repeat, read later. Add the total to the ledger and show
it. While the lookups run you may start Research.

## 5. Follow the lookups

- **`prospects.enrichStatus` present:** call it with the `jobIds` (1-100). Per job `outcome`: `searching` (ask
  again in a minute), `found` (the address is on the person), `not_found` (not charged), `no_credits` (the wallet is
  empty: stop and tell the person), `conflict` (the address belongs to another person in Dealmaker; a person decides
  which lead keeps it; it was charged), `failed` (`error`). `done: true` when none is searching.
- **Not present:** wait a few minutes, then read `prospects.get` (or `prospects.search` for the whole list) once,
  not in a tight loop. A lookup can take a few minutes. No email after about 10 minutes most likely means none was
  found; you cannot tell "not found" from "failed" this way. `prospects.get` may also carry `lastEnrichment`, the
  latest lookup and how it ended.
- The chat cannot wait that long: say "The lookups are running; tell me 'continue' in a few minutes." That is not a
  decision, just a nudge.
- Many people with no email at once: tell the person to check the company's credits and whether the owner allowed
  platform data.

## 6. Show the result

"Found 17 emails out of 21: 14 valid, 2 risky, 1 unknown. 4 people have no email; they stay LinkedIn only. Spent so
far: up to 1,137 of 1,700 credits (~$17.06); left: 563." Then a table: name, company, email, status.

| `verifiedStatus` | Emailed by the campaign? |
|---|---|
| `valid` | Yes |
| `risky` | Only if the workspace allows risky emails (off by default; no tool shows or changes it) |
| `unknown`, or no email | No: LinkedIn steps only |

Never mark an email valid yourself. `kind: "verify"` re-checks an address the person already has; it never looks for
a new one.

## Done

The same list, some people with a work email and a status. Progress report, then Research.

| Call | Kind | Key args | Key answer |
|---|---|---|---|
| `enrichment.status` | Read | none | `status`, `providers[].kinds` |
| `prospects.enrich`, dry run | Free | `prospectId`, `kind` | `quote` or `status: unavailable` |
| `prospects.enrich` | Paid only for an email found | the same | `jobIds` |
| `prospects.enrichStatus` (if present) | Read | `jobIds` | per job `outcome`, `done` |
| `prospects.get` | Read | `id` | `emails[]` (`value`, `verifiedStatus`), `lastEnrichment` |
