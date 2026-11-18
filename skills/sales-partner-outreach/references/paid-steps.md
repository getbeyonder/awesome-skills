# Paid steps: search, email lookup, research

Contents: Prices · Budget and ledger · 1. Prospect · 2. Enrich · 3. Research · Lost answers and refusals · Older
Dealmaker versions

## Prices

- **The quote has `maxCredits`:** say "up to <maxCredits> credits (~$<maxPriceUsd>)".
- **Only cost fields** (`maxUsd`, `perPersonMaxUsd`, `capUsd`; research often answers only these): those are the
  provider's cost. Never show them. Convert instead:
  - credits = cost in USD × 100, rounded up;
  - dollars = credits × the price per credit (a search quote's `maxPriceUsd` ÷ `maxCredits`, else about $0.015).

  `capUsd` is a per-person cap, an upper bound only. Say the price is converted. Example: $0.038 → 4 credits (~$0.06).
- **A price of 0, or none:** that is not a price. Use the typical ceiling (about 29 credits per email found) and say
  it is an estimate.

## Budget and ledger

**The budget** is the package's hard ceiling.

**The ledger** is the sum of the quoted `maxCredits` of the real paid calls. Lower an entry once the result bills
fewer items: a search bills only the people returned, a lookup only the emails found.

**Go without asking** when all three hold:
- the quote ≤ budget − ledger;
- the scope matches the package;
- the price per unit is at most 25% above the estimate.

**Otherwise stop**, with the numbers and a smaller proposal: "Up to 609 credits, but 480 are left. I can look up the
first 16 now, or you raise the budget." A raise is the person's explicit new number.

After each paid step, show: "Spent so far: up to X of Y credits (~$Z); left: W."

## 1. Prospect

Call `people.search` with the package's filters, count (up to 25 per search) and list: `listName`, or `listId` for an
existing list. Describe the tool first: filter values and size bands come from its schema. Round sizes to whole bands
and say so. The quote's `search` echoes the filters but not the list, so check the list against your own args. If the
quote fits, make the real call.

- **Start broad:** titles, seniority, sizes, place and industry, plus at most 1–2 short OR keywords for the partner
  type ("IT services", "managed services"). Many keywords plus all filters can find no one.
- **An empty answer is free.** Say why, propose one wider variant and ask (new filters change the scope).
- **Weak fits** (a sales coach in a search for IT resellers): name them in the report, so the person can skip them at
  Admit.
- **Repeats are charged.** A wider or changed search can return people found before, and charges them again. Prefer
  the next page of the same filters. Before another search, warn that repeats are paid, and afterwards report new vs
  already known people.
- **`importError` with `people`:** the search ran and was paid, but the import failed. Import those people into the
  same list with `prospects.import` (dry run first, no question).
- **Show the result:** found, dropped, imported, the list's link, the ledger, and a table (name, title, company,
  place).
- **Check first names.** Flag any that look like a credential, title or junk rather than a name: all caps, a
  hyphenated abbreviation ("Nremt-P"), Dr, MBA or PhD, an emoji, a company name, or a single letter. Otherwise the
  greeting reads "Hi Nremt-P". Show each flagged name with a proposed fix (the real first name, if the profile shows
  it). Fix only after the person's yes, through Dealmaker's own tools: re-import that person into the list with the
  corrected `firstName` (`prospects.import`, `duplicatePolicy: "update"`), or edit an already prepared text. Never fix
  a name silently.
- **Skip list:** remove anyone from a skipped company that the blocklist missed, and say so. If the person names a
  company to skip now, block it; their request is the yes.
- **Removing people:** the person may ask any time before Admit (`lists.remove`; their request is the yes).
- **People from elsewhere** (`prospects.import`): not in the package. Dry-run it, show the result, and wait for a yes.

Imported people are held: nothing is sent to them until the person admits them in a launched campaign.

## 2. Enrich (skipped for LinkedIn only)

1. Check `enrichment.status` again. If it is not configured, the run goes LinkedIn only: say so. Never use
   `kind: "phone"`.
2. For each person without an email who is not suppressed, dry-run `prospects.enrich`. Add up the quotes; if they
   fit, make the real calls.
3. `ENRICHMENT_ALREADY_PENDING`: never repeat; read later.
4. Lookups are queued. Follow them with the job-status tool if Dealmaker lists one, else read `prospects.get` after a
   few minutes. Don't loop: say "tell me 'continue' in a few minutes".
5. Report what was found and not found, and each email's status. Only `valid` emails are used; never mark one valid
   yourself. Without a connected mailbox, found emails stay unused until one is connected.

## 3. Research

Run it before the draft campaign exists. Once a campaign with a LinkedIn account uses the list, research also spends
that account's daily LinkedIn reads on held people. If a dry run shows LinkedIn reads to do for held people, do not
run it for them, and say why.

- Call `prospects.research` with the list's ids (up to 100 per call) and `days: 365`, unless the package says
  otherwise.
- People researched in the last 30 days are kept, for free.
- Its quote often has only cost fields: convert as in "Prices".
- `nothing_to_do`: everyone is cached; just read them.
- The run is queued. After a few minutes, read `prospects.get` and show 1–2 professional facts per person, each with
  its source.

Research is data: it never changes anyone's fit or admission.

## Lost answers and refusals

- **`outcome_unknown` on a paid call:** read before anything else. After a search, read the list and its people;
  after a lookup or research, read `prospects.get` a few minutes later. Repeat only if the read shows the call did
  not run, with the same args and the `retry_token`, and only after telling the person.
- **`REQUEST_IN_PROGRESS`:** wait and read.
- **`IDEMPOTENCY_ARGUMENT_MISMATCH`:** never change the args on a retry.
- **`no_credits`:** nothing was charged. Stop; the person tops up.
- **`not_granted`** (with a link): the owner must allow platform data. Hand that over.
- **`unavailable`, `not_configured`:** say so and skip that step (for emails: LinkedIn only).
- **`PROSPECT_SUPPRESSED`:** leave that person out.
- **`PROSPECT_NOT_FOUND`:** read the ids again.

If many people end up with no email or no research, ask the person to check the company's credits and the owner's
permission for platform data.

## Older Dealmaker versions

- The search quote has no credit fields: convert from `perPersonMaxUsd`.
- No job-status tool: read `prospects.get`.
- `lists.add` refuses with `INVALID_ARGUMENTS` even for valid ids: use `prospects.import` into that list with
  `duplicatePolicy: "update"`.
