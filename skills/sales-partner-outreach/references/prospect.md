# Phase 1: Prospect

Find people who match the partner profile (Setup Q5) and save them into a Dealmaker list. They have LinkedIn links
and no emails yet. Only Dealmaker is used. You never call the people-data provider yourself: Dealmaker asks the
platform's people search.

Input from the approved package: the filters, the searches and their counts, the list name, the skip list (already
in the blocklist), the budget and the estimate per person. Nothing is asked here unless a stop condition comes up
(SKILL.md, "When you stop and ask").

## 1. The plan comes from the package

Use the filters, counts and list name exactly as the package showed them (e.g. `US partners: IT agencies 11-200`).
Dealmaker does not store the filters, so the list name is their only trace. No recap, no "OK?".

A company the person names only now (not in the package): block it with `blocklist.add` (as Setup C2); their
request is the yes for that one value.

## 2. The profile becomes the filters

`app_tool_describe(warmleads, people.search)` first: the fixed values and limits below may differ by version.

| Profile part | Filter | Notes |
|---|---|---|
| Who in the company | `seniorities`, `titles` | `seniorities` only: owner, founder, c_suite, partner, vp, head, director, manager, senior, entry. "Founders and owners" → owner, founder, partner. Exact titles in `titles` (up to 25). |
| Partner type | `industries`, words in `titles` | Free text, up to 25. |
| Where | `locations` | "United States" or "Texas, United States". US only. |
| Size | `companySizes` | Bands only: 1-10, 11-50, 51-200, 201-500, 501-1000, 1001-5000, 5001-10000, 10001+. Round to whole bands and say so. |
| Must-have words | `keywords` | One string; it may match the job title only. |

At least one filter. `count` 1-25 (default 25). `page` 1-100. For more people, prefer the next page of the same
filters (`nextPage`); a page past the end answers `found: 0, more: false` and is free. Further pages may be refused
on some platforms: then run another search with different filters (another state, title or size) into the same
list. A changed or wider search (page 1) can return people an earlier search already found, and each person
returned is charged again: before such a search, tell the person that repeats are possible and are paid. Only the
searches the package listed run without asking; a new one is a scope change. `listName` (found, or created) or
`listId` (an existing list), never both.

## 3. Dry run: the quote

`app_write(warmleads, people.search, args, dry_run: true)`. Nothing is searched or charged. Dealmaker's answer
(often inside `appPreview`):
- `status: "ready"`, `search` (the filters exactly as they will be sent), and `quote`. `search` may leave out
  `listName` / `listId`: check the list name or id against the `args` you sent, not against `search`.
- `quote.maxCredits` and `quote.maxPriceUsd`: the price is "up to <maxCredits> credits (~$<maxPriceUsd>) for up to
  <maxPeople> people".
- No credit fields (an older Dealmaker): convert from `perPersonMaxUsd` as in `enrich.md` ("Showing a price"), say
  it is an estimate. Never show `maxUsd`.
- Only people returned are billed, repeats from an earlier search too; an empty answer is free; the same search
  within 7 days comes from a cache.
- `not_configured`: Dealmaker has no connection to the platform's search; say so and stop.

**Go or stop** (`enrich.md`, "The budget and the ledger"): `search` matches the package's filters and count, the
list name or id in your `args` is the package's, the quote fits what is left of the budget, and the price per person
is at most 25% above the package's: go on without asking. Anything else: stop and ask.

## 4. The real search

The same `args`, without `dry_run` (`confirmed` / `retryKey` only if the schema lists them). Dealmaker's answer:
- `found`, `dropped` (`noLinkedin`, `noLastName`: never guess those people), `more`, `nextPage`.
- With a list: `list` {`id`, `name`, `created`, `link`}, `imported` {`created`, `updated`, `skipped`, `refused`,
  `added`, `alreadyInList`, `held`}, `issues` [{`row`, `name`, `outcome`, `reason`}]. People already in Dealmaker are
  updated, not duplicated. Blocked, suppressed or rejected people are not added.
- `importError` with `people`: the search ran and was paid, but the import failed. The people are not lost: pass
  `people` as the `rows` of `prospects.import` into the same list (dry run first; it is free and part of the approved
  search, so no question).
- Never holds emails or phones.

**Imported people are held** ("Waiting for admission review"). Nothing is sent to anyone until the person admits
them in a launched campaign. No tool admits people.

## 5. Show the list (no waiting)

The search already imported the people into the list; there is no review step before Enrich. Show the result, then
go on:

"Found 23 (2 dropped: no LinkedIn link). Saved to the list '<name>' → <list.link>. Spent so far: up to 644 of 1,700
credits (~$9.66); left: 1,056." Then a table: name, title, company, country.

After a follow-up search, say how many are new (`imported.created`) and how many were already known
(`imported.updated`): "Found 5: 2 new, 3 already in the list from the earlier search. All 5 are charged." The ledger
counts every person returned.

- **Skip list.** A person from a company on the package's skip list that the blocklist did not catch: remove them
  from the list (below) and say so. That is inside the package.
- **Removing people.** Say once: "If you want anyone out, tell me their names any time before you admit them; I'll
  take them off the list. Or remove them yourself on the list screen: <list.link>. I'm going on with the email
  lookup now." Removing someone before Enrich also saves their lookup.

To remove people:
- `app_read(warmleads, prospects.search, { filters: { listId }, limit: 100 })` for the ids. Filters apply within a
  page: follow `cursor` until `done`.
- `app_write(warmleads, lists.remove, { listId, prospectId })`, one per person, after the person names them (their
  request is the yes). `changed: false`: they were not in the list. The person stays in Dealmaker; only the list
  membership goes. Once a draft campaign uses the list, removing someone stops them in it; that is fine before Admit.

To add a person already in Dealmaker to a list (e.g. one removed by mistake): `lists.add` (`listId`, `prospectId`).
It can refuse with `INVALID_ARGUMENTS` even with valid ids. Then use `prospects.import` with that `listId`, one row
per person (`firstName`, `lastName`, `linkedinUrl` from their record) and `duplicatePolicy: "update"`: Dealmaker
matches the existing person and answers `created: 0` with `updated` and `added` per person; no duplicate. Dry run
first, as for any write.

## 6. People the person brings from elsewhere

`prospects.import` for a CSV or a list they already have. Not in the package: free, but it writes many records, so
dry run, show it, and wait for the person's yes. `listName` the first time, `listId` after; up to 100 `rows` per
call; each row needs `linkedinUrl` or `email` (`emailStatus` only if the person knows it was verified). Report
created, updated, skipped, refused and why. These people are held too. Before relying on an imported email, Enrich
can verify it (`kind: "verify"`).

## Done

A Dealmaker list of held people with LinkedIn links. Progress report (done / cost / next), then Enrich (or Research
when the package chose LinkedIn only).

| Call | Kind | Key args | Key answer |
|---|---|---|---|
| `people.search`, dry run | Free | filters, `count`, `listName` / `listId` | `search`, `quote` |
| `people.search` | Paid | the same | `found`, `list`, `imported`, `issues` |
| `prospects.search` | Read | `filters.listId`, `cursor` | `items[]._id`, `done` |
| `lists.remove` | Write | `listId`, `prospectId` | `changed` |
| `lists.add` | Write | `listId`, `prospectId` | refused: `prospects.import` with `duplicatePolicy: "update"` |
| `lists.list` | Read | `limit` | lists, to check after `outcome_unknown` |
