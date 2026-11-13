# Phase 0: Setup

One conversation at the very start, before anything is searched or paid. You read the board yourself, ask every
question of the run once, in one message, run every dry run and quote the package needs, and show **one approval
package**. One explicit yes to it lets you run the rest of the flow on your own. Nothing in Setup costs credits.

```
prompt → A. read-only checks → to-do for the person → B. one message with 17 questions (each with a proposal)
       → follow-up round (only if needed) → D. dry runs and the search quote → E. the approval package
       → one "yes" → C. writes in order (suite → blocklist → Booking → AI CRM), each re-checked by its dry run
       → progress report → Phase 1 at once
```

## Conversation script

Each point is one message from you (or one short exchange).

1. **Greeting.** "Before we search for anyone, I'll set up the run once: I check your board, ask all my questions
   in one message, and show you one package to approve. After your yes I run it on my own and come back to you only
   for your clicks in the apps, and if something does not fit the package. Nothing is paid in this part."
2. **Checks** (Step A). "I'll look at your board first. This only reads; nothing changes and nothing is paid." Run
   A1 to A6, then say what you found in plain words, never as JSON. A check that fails (no answer, a refusal): say in
   one line which check could not be done, and go on. Do not guess its answer.
3. **To-do** (A7). Only the missing items, each with its link and why it matters. No answer needed now.
4. **One message with all questions** (Step B). The person may answer "ok" to all, "ok, except 5 and 17: ...", or
   each one. "ok" accepts the proposals; it is not the yes to the package.
5. **Follow-up, only if needed:** answers missing, unclear or failing a check. One message, only the open points,
   each with its reason and a new proposal. Never ask again what the prompt said, what Step A read, or what the
   person already answered. After this round an open question gets its default (or "later"), and the package says
   so. A question with no default that blocks a phase (Q5 for Prospect) stays open; the package cannot be shown
   without it.
6. **Dry runs** (Step D). "Now I check exactly what each change would do and what the search costs. Still nothing
   changes and nothing is paid." No question in this step.
7. **The approval package** (Step E). One message. Wait for one explicit yes.
8. **Writes** (Step C), right after the yes, in order, without a new question.
9. **Done.** A progress report, then Prospect at once: "Setup is done. Applied: <...>. Spent: 0 of <budget>
   credits. Still on your to-do: <...>. Next: I search for partners now."

## Step A. Read-only checks

### A1. Apps on the board

- `list_apps`: the apps, their titles, whether each is ready, and their `url` (for screen links). There is no
  "ready" flag: read each app's fields. An app with `state: "ready-made"` and `deployment: "bound"` is installed
  and usable; if those exact fields are missing, a bound deployment means usable. Anything else (installing,
  waiting for the owner, archived) is not ready yet.
- Dealmaker, AI CRM and Booking all present and ready: say so.
- Some missing: look in `catalog_list` for a suite whose apps are Dealmaker, AI CRM and Booking (for example one
  called "Sales partner search"). If there is one, `suite_get` shows which of its apps are present, getting ready,
  waiting for the owner, or archived before. Never promise a suite the catalog does not list.
- Say in one line: "Your board is missing <apps>. The suite '<title>' installs Dealmaker, AI CRM and Booking
  together, with a meeting type for partner calls and a partner pipeline. Free, no credits. I'll put it into the
  package with its exact dry-run result." The install itself waits for the package's yes (Step D1, Step C).
- Apps archived before: the suite's dry run asks whether to restore their data or start fresh. Put that choice as a
  question into Step B (your proposal: restore); you do not choose.
- The person does not want the suite: nothing is installed. Say what it means ("Without Booking there is no booking
  page; without AI CRM meetings are not logged") and offer single apps only if the person wants them (then they go
  into the package instead). Without Dealmaker the flow cannot run: say so and stop.
- **Dealmaker is missing:** A2, A3 and A6 cannot be read yet. Ask the questions anyway; the package marks what is
  "checked right after the install".

### A2. Sending accounts

`app_read` Dealmaker `senders.list` (`limit` 50). Look for:
- **LinkedIn:** `kind: "linkedin"`, `status: "connected"`. Without one there are no LinkedIn steps, and Dealmaker
  cannot read people's recent posts.
- **Mailbox:** `kind: "email"`, `status: "connected"`. Without one, emails found later cannot be used: Q8 proposes
  LinkedIn only, unless the person commits to connect a mailbox before Outreach.
- `personStep` (`connect`, `reconnect`, `review_restriction`): a step only the person does, in Settings → Senders.
- **Capacity:** each sender's `limits` (for example the weekly invitation limit in effect and what is
  `availableToday`) and its other active campaigns (`campaigns.list`: active campaigns that use this account share
  its limits). Report them; the package states the realistic timeline from them (e.g. 2 invitations a day while
  warming up → 10 people ≈ a week).
- The mailbox's sender name, postal address and DNS (SPF, DKIM, DMARC) are **not** visible to you. Dealmaker will
  not send an email without them. The person checks them (to-do).
- A member sees only their own accounts; an owner or admin sees everyone's. A campaign may use only its owner's
  accounts. Ask the person which accounts are theirs (Q9); Outreach checks it again.

### A3. Email lookup

`app_read` Dealmaker `enrichment.status`. `kinds` has `email`: on. `status: "not_configured"`: say "Email lookup is
not set up for your company. That is a platform setting, not something you change in Dealmaker. I'll plan LinkedIn
only for now." Draft Q8 as LinkedIn only.

### A4. Booking: calendar, meeting type, hours

- `list_calendars`: `connected: true` is enough.
- Not connected: `connect_calendar` (a read; it only makes the person's own link). `available: false`: the
  platform's Google connection is not set up for this Booking; free times then ignore the calendar (a note, not a
  to-do). No link comes back (the host blocked the call, or a refusal): never retry around a host's denial; the
  to-do gives the click path instead, and you fetch the link again before the first booking link.
- `list_types`: a partner intro type that is `active`, and its `hostMembers` (`you: true` is the person). It counts
  as found only if its name or slug contains "partner": then Q15 only confirms it. Another active type (e.g. a
  generic "Intro call"): Q15 offers "reuse '<that type>' or create 'Partner intro'". None: Q15 proposes one, created
  in C3.
- `get_schedule`: the person's time zone and hours, for the Q14 proposal. A time zone of "UTC" is probably an unset
  default, not a choice: cross-check it with the sender's `timezone` from `senders.list` (A2), and in Q14 ask the
  person's city.

### A5. AI CRM pipeline

`list_pipelines`: the partner pipeline, its stages with ids, and `meetingStages`. Usually it logs meetings but does
not move the deal on booking until `meetingStages` is set (Q16).

`meetingStages` is per pipeline: it moves only deals that are in that pipeline. Before you propose a deal move, find
out which pipeline a Booking guest's deal lands in: read it if a tool shows it (for example a default pipeline in
`list_pipelines`), else ask it in Q16. Propose the move on that pipeline. A rule on another pipeline (or on a new
one) does not fire for booked deals that land elsewhere; say so.

### A6. Company profile (only when a campaign already exists)

`campaigns.list`; if there is a campaign, `knowledge.get` with its id, and read only `company`. The profile is
workspace-wide, so any campaign will do. An empty `companyName` or `description` goes on the to-do: Dealmaker's AI
writer writes nothing without it. Not empty: once Q1 is answered, compare the description with it. It describes a
different product: suggest updating it (`<origin>/settings/workspace`) and put that on the Launch to-do, since the AI
writer uses it in every message. No campaign: skip; Outreach checks it.

### A7. The to-do

Only what is really missing, numbered, in one message, each with what to open, what to do, and why:
- **No LinkedIn** (or `personStep: "connect"`): "Connect your LinkedIn account in Dealmaker:
  <origin>/settings/senders → add a LinkedIn account. Only you can do this. Without it the campaign cannot send
  LinkedIn invitations, and Dealmaker cannot read partners' recent posts."
- **LinkedIn needs a step** (`reconnect`, `review_restriction`): "Your LinkedIn account '<label>' needs you:
  <origin>/settings/senders → reconnect it (or review the restriction LinkedIn shows). Until then it sends nothing."
- **No mailbox:** "Connect a mailbox on the same screen. Fill in the sender name and your company's postal address:
  US law for commercial email (CAN-SPAM) needs both in every email, and Dealmaker will not add a mailbox without
  them. Then check that its DNS (SPF, DKIM, DMARC) shows 'pass'. Without a mailbox the campaign can still run on
  LinkedIn only."
- **A mailbox is connected:** "Please open <mailbox> in <origin>/settings/senders and check three things: a sender
  name, a real postal address, and DNS 'pass'. I can't see them, and Dealmaker will not send an email without them."
- **No calendar:** "Connect your Google calendar to Booking: <link from connect_calendar>. Open it while signed in
  and allow Google Calendar in Google's own window. Partners then see only your free times. I never see your
  calendar." No link came back: "Connect your Google calendar in Booking's calendar settings (<booking origin>).
  I'll fetch the direct link again before the first booking link."
- **Empty company profile:** "Fill in your company's name and a short description in Dealmaker:
  <origin>/settings/workspace → Company information. Dealmaker's AI writes no message without them."
- **Company profile describes another product** (after Q1): "Your company profile in Dealmaker describes <...>, not
  <product>. The AI uses it in every message; please update it: <origin>/settings/workspace → Company information."

End with: "You can do these now or later, any time before Launch; just tell me 'done'. I check again before the step
that needs each one."

## Step B. The 17 questions

Rules: one message; only what is missing (the prompt often gives the place and the product's name); a proposal
with every question where you can make one. Never name an internal tool, field or id in this message. Proposals come
from the prompt and Step A, never invented. Where you cannot guess (Q1, Q3), say so. A question Step A answered is
left out or turned into a confirmation.

### The message (adapt the proposals)

```
A few questions for the whole run. Each has my proposal: answer "ok" to keep it,
or write only what you want changed (e.g. "5: only Texas; 17: 1,000").
After this I show you one package to approve; nothing happens before that.

A. Product and offer
 1. <Product> in 2-3 sentences: what it is, who buys it, typical deal size? (I can't guess this one.)
 2. A partner gets <a share, e.g. 20% of the first-year contract> per client they bring;
    we ask for a 15-minute call. OK?
 3. Your demo video link (public https, e.g. a Loom share link)? Or "later".
 4. Partners' common questions, and what never to say. My draft: <2-3 Q&As>.
    Never: exclusivity, a fixed income, a price for the client.
B. Who to find
 5. <Founders, owners and managing partners> of <partner type> in the US, <11-200> employees. OK, or change?
 6. Companies to skip (current partners, clients, competitors)? Their websites, e.g. acme.com. Or "none".
 7. How many people: 25 as a test?
C. Channels
 8. <With a mailbox:> Email first, then LinkedIn, 3 follow-ups; LinkedIn only for people without a valid email.
    Or LinkedIn only for everyone?
    <No mailbox connected:> LinkedIn only (an invitation, then an opener and 2 follow-ups), since no mailbox is
    connected; so no email lookups are paid. Or will you connect a mailbox before the campaign is made?
 9. Send from LinkedIn '<label>' and mailbox <address>. Are these your own accounts?
10. Campaign "<US partners: type size (channels)>". OK? (Texts are always in English: Dealmaker's AI writes English only.)
11. Messages go out automatically as Dealmaker's AI writes them, from your product, offer and never-say lines.
    Or do you want to approve every message yourself? (Then anything you don't approve in time is not sent,
    which can leave LinkedIn messages unsent.)
D. Conversation
12. When a partner replies, should the AI draft the answer only from your product, offer and FAQ?
    (You switch it on at launch; you approve every reply.)
13. If a partner goes quiet, draft a nudge after 3 days and one after 7 (you approve each)?
    I suggest no for the first run.
E. Calls
14. You take the calls, Mon-Fri 10:00-17:00 in your time zone (<zone from Booking>).
    <If Booking says UTC: Booking shows UTC, which is probably not set yet; your sending account says <zone>.
    Which city are you in?> You connect your Google calendar yourself. OK?
15. Partners book "<Partner intro call>": 30 min, video, the form asks about their clients and the topic. Keep it?
    <Only a generic type exists: Reuse your "<Intro call>" for partners, or create "Partner intro"? I suggest
    creating it.>
16. When a partner books, move their deal in AI CRM to "<Call booked>", and to "<Call held>" after the call?
    <Booked deals land in "<pipeline>" | Which AI CRM pipeline do booked deals land in?>
    <No fitting pipeline: or create a new pipeline "<Partners>" with these stages?>
F. Money
17. For <25> people the whole run costs about <1,550> credits (~$<23.25>).
    Your budget, a hard ceiling: <1,700> credits (~$<25.50>)? I spend within it without asking again,
    and stop and ask before going over it.
```

### Each question

| Q | Proposal from | What you check | If skipped / "later" | Used in |
|---|---|---|---|---|
| 1 Product | Nothing (you may quote the company profile's description: "Is this still right?") | What it is, who buys it, deal size (a range is enough). No links, emails or phone numbers. Up to 1500 characters. "AI stuff" or "see our website" fail. | Ask once more; needed by Q5, Outreach templates, the knowledge base | Templates `instructions`; `knowledge.save` `product` |
| 2 Offer and ask | A common reseller offer (a share of first-year revenue) and the smallest ask (a short call); say it is only a proposal | Both parts, the partner's part as a number or clear rule. A part left as a placeholder (`[commission]`, `[payout terms]`): see SKILL.md, "Placeholders never reach the apps". Nothing that contradicts Q4. No links. Up to 1000 characters. "A good commission" fails. | "ok" keeps the draft | Templates `instructions`; `knowledge.save` `offer` |
| 3 Demo video | Nothing | Starts `https://`, a real domain, no user or password, a share link that opens without sign-in (ask the person to make sure; you do not open it) | Allowed; until there is a link, never promise a video in a reply | Added to reply drafts in Engage; kept in the chat only (the knowledge base refuses links) |
| 4 FAQ and never-say | 2-3 questions every reseller asks (what do I do, when am I paid), answered from Q1-Q2; never-say lines for what was not offered | Each FAQ item a question with an answer. At most 12 items (question 200, answer 600 characters), at most 15 never-say lines of 200; Q1+Q2+Q4 up to 6000 characters. No links, emails, phones. Agrees with Q2. | Keep your never-say lines (the safe side) and the draft FAQ; the package says so | Templates `instructions`; `knowledge.save` `faq`, `neverSay` |
| 5 Partner profile | From Q1 (who already sells to the same buyers) and the prompt; five parts: partner type, who in the company, where, size, must-have title words | Maps to `people.search` filters (see `prospect.md`). US only: a place in the EU, the UK or elsewhere is outside this flow; say so. Sizes rounded to whole bands, and you say so. At least one filter. "Anyone" fails. | No default: blocks the package | `people.search` filters; the list name |
| 6 Exclusions | Nothing; ask for websites | Each company as a domain (`acme.com`: strip `https://`, `www.`, paths and show the result). A person as a work email or LinkedIn `/in/` link. A name without a domain goes to the follow-up; never guess a domain. Note up to 500. | "None" | C2 `blocklist.add` |
| 7 How many | 25 (one search's maximum) | A whole number ≥ 1. Above 25: plan the searches (different filters each, e.g. by state) and list each in the package; above 100: research in several calls. Fits under Q17. | 25 | The searches; the estimate |
| 8 Channels | Email + LinkedIn only when lookup is on **and** a mailbox sender is in `senders.list` (or the person commits to connect one before Outreach); otherwise LinkedIn only (say why) and no email lookups in the budget | One of the two. "Email only" is not planned in this flow (people without an email would get nothing, and posts are read only through LinkedIn); explain and propose email + LinkedIn. No other channels. | The proposal | Enrich runs or is skipped; `create_draft` `channels`, `steps` |
| 9 Accounts | The connected accounts from A2; if several people's are visible, list them and ask which are theirs | One LinkedIn account; for email one or more mailboxes; each connected with no `personStep`, or on the to-do. "Use a colleague's LinkedIn" fails: a campaign uses only its owner's accounts. | Prospect, Enrich, Research can run; Outreach stops and asks | `create_draft` `linkedinSenderId`, `emailSenderIds` |
| 10 Name | Name from Q5 and Q8. Texts are always in English (Dealmaker's AI writes English only). | Name 1-150 characters. | The proposal | `create_draft` `name` |
| 11 Review mode | `auto`: each written text goes out when due, without per-message approval | `auto` or `manual`. With `manual` the person approves every message on the Queue tab; a text not approved in time is not sent, which can leave LinkedIn messages unsent; that adds a step to the Launch moment. "You approve them for me" fails: only the person approves. | `auto` (the package shows it plainly) | `create_draft` `reviewMode` |
| 12 AI dialogue | A question; say what on and off mean | Yes or no. Yes needs a saved knowledge base, which you save right after the campaign is created. | Off | The launch hand-off, link 1 |
| 13 Nudges | No for a first run | Yes or no. At most 2 per silence, 4 per conversation. | Off | The launch hand-off, link 1 |
| 14 Calls | Hosts from the meeting type (`you: true`), weekdays, office hours, the time zone from `get_schedule`; "UTC" there is probably unset: cross-check the sender's `timezone` from `senders.list` and ask for the city (also when there is no zone) | Each host an active admin or member of Booking (`list_members` for another person); 1-20 hosts; days 1 (Mon) to 7 (Sun), `HH:MM`, from before to; a real IANA zone. Only a Booking admin changes another member's hours. The calendar is always each host's own step. | Booking keeps its hours; no hours change in the package | D3 / C3 `set_schedule`; `create_type` / `update_type` `hosts` |
| 15 Meeting type | The type A4 found whose name or slug contains "partner" ("Keep it?"); only another type exists: "reuse '<existing type>' or create 'Partner intro'"; else propose: "Partner intro call", slug `partner-intro`, 30 min, video, two form questions (clients; topic) | `name` and `slug` (lowercase letters, digits, dashes, 1-50); `durationMin` 5-720; `location.kind` video, phone, address or custom; at most 10 questions with a lowercase `key`, a `label`, a `type`. No `price` or refund window: those are the person's. | The existing type; if missing, the proposal | D3 / C3 `create_type`; the personal booking links |
| 16 Deal move | The "call booked" / "call held" stages of the pipeline booked deals land in (A5; ask if unknown); no fitting pipeline: a new one ("New pipeline" in D4 / C4) | Each stage exists, or is in the new pipeline's plan; booked deals land in that pipeline (else its rules never fire: say so); a stage with required fields is refused; only an AI CRM admin may change it | No move; meetings are still logged | D4 / C4 `save_pipeline` (`meetingStages`; no `id` for a new pipeline), `save_pipeline_stage` |
| 17 Budget | A hard ceiling a little above the plan for Q7 (about 10% headroom): search ~28, emails ~29, research ~5 credits a person (leave out emails for LinkedIn only) | A number. Dollars become credits at about $0.015 a credit, shown back. Below the plan: say what fits and propose a smaller count or LinkedIn only. "No limit" fails: the package needs a ceiling. | The proposal | The package's budget; the ledger; never sent to a tool |

Defaults you do not ask about (the package shows them): look up emails for everyone found without one; research
the whole list; news from the last 12 months; no test run to a colleague. The person may ask for one: if
`lists.list` shows an existing internal one-person list (a test receiver), ask in Setup whether to reuse it; else
the package lists a new one-person test list. Either way the test gets its own small draft. With LinkedIn steps,
people already connected to the sender are left out by default, so a colleague who is already a connection gets
nothing on LinkedIn: the package says so and offers to include connections for the test draft only.

## Step D. Dry runs before the package

Everything the package will promise is checked first, so the package shows real results, not guesses. Each call
here is a read or a dry run: nothing changes and nothing is paid. Run only what the answers need.

### D1. Suite install (only if apps are missing)

`suite_install` with `dry_run: true`. Note what it would install, which links between the apps it would switch on,
and, for apps archived before, the person's restore-or-fresh answer from Step B. `needs_owner` or `payment_required`:
the install is the owner's step; give the link, leave the suite out of the package and say what cannot run.

### D2. Blocklist (only if Q6 named values)

`blocklist.list` for values already blocked, one call per `kind` the values use (`company`, `domain` or `person`;
the tool needs `kind`). Values already blocked are left out: "already blocked". `blocklist.import` with
`dry_run: true` when present (each row: ready, duplicate or invalid); else `blocklist.add` with `dry_run: true` per
value if it supports it; else the exact list of values is the preview. An invalid value goes back to the person
before the package.

### D3. Booking

- Meeting type missing, Q15 chose a new one, or host wrong: `create_type` (or `update_type` with `hosts`) with
  `dry_run: true`. Note the name, address (`/b/<slug>`), length, place, hosts and form questions from the preview. A
  non-admin is refused: only a Booking admin can create it; it goes into the package as the person's step instead.
- Hours changed by Q14: `set_schedule` with `dry_run: true` (`timezone`, `weekly` for the whole week,
  `overrides`). Note the new hours, today's hours, and `bookedOutside` (meetings already booked outside the new
  hours; they stay booked).

### D4. AI CRM deal move (only if Q16 is yes)

`list_pipelines` for the pipeline and stage ids. `save_pipeline` with `id`, `meetingStages` and `dry_run: true`
(e.g. booked and rescheduled → the call-booked stage, held → the call-held stage; statuses left out stay as they
are). If it gives no preview of its own, the exact change is the preview.

**New pipeline** (Q16 chose one): `save_pipeline` without `id` creates it, with the standard stages unless
`standard: false`. Its dry run returns only the platform's generic preview, no app preview: the preview is the
exact name and stages you will send. `save_pipeline_stage` needs the new `pipelineId`, and `meetingStages` can only
be set as a change on an existing pipeline, so neither can be dry-run before the pipeline exists. The package shows
the name, the planned stages and the deal move, and marks the stages and the deal move "checked right after
creation".

### D5. The search quote

`app_tool_describe(warmleads, people.search)`, then `people.search` with `dry_run: true`: the Q5 filters, the Q7
count (up to 25 per search) and the proposed `listName`. Nothing is searched or charged. From the quote: the price
per person (`maxCredits` / `maxPeople`) and the filters exactly as they will be sent (`search` echoes only the
filters, not `listName` / `listId`: check those against the `args` you sent). Several searches: quote each.
`not_configured`: Dealmaker has no connection to the platform's search; say so: the run cannot find people.

### D6. The estimate

- Search: the quote(s) from D5.
- Emails (email + LinkedIn only): the number of people × about 29 credits per email found (the typical ceiling; the
  real quote needs the people, so it comes in Enrich).
- Research: the number of people × about 5 credits.
- The sum is the estimate. It must fit under the budget (Q17); if not, propose less before the package.

### Dealmaker missing (the suite is in the package)

D2 and D5 cannot run before the install. The package shows those lines from the answers and marks them "checked
right after the install". After the yes and the install, run them; any result that differs from the package (a value
refused, a quote more than 25% above the typical price, no connected account) is a stop.

## Step E. The approval package

One message, in plain words, no internal ids or field names. Mark lines that used a default ("(default)") or
"later". Leave out lines that do not apply.

```
Approval package: one "yes" lets me run all of this on my own

WHAT I SET UP NOW
Board          Dealmaker, AI CRM, Booking: on the board
               | install the suite "<title>": <apps>, and switch on <links between them>. Free.
                 (dry run: <what it would install; restore or start fresh>)
Skip list      block <acme.com, beta.io> in Dealmaker, so no campaign reaches them | none
Booking        meeting type "<name>" at /b/<slug>: <30> min, video, hosted by <you>, <2> questions on the form
               (dry run OK) | keep "<name>"
               your hours: <Mon-Fri 10:00-17:00, zone> (today: <...>; <n> booked meetings fall outside and stay
               booked) | unchanged
AI CRM         pipeline "<name>" (booked deals land here): when a partner books or reschedules, the deal moves
               to "<stage>"; after the call is held, to "<stage>"
               | create the pipeline "<name>" with stages <...>, then the same deal move (dry run: create only;
                 stages and deal move checked right after creation) | no move (meetings are still logged)

WHO I FIND
People         <seniority> of <partner type> in <US place>, <size bands>, <keywords>
               search 1: <filters>, <25> people [search 2: <filters>, <n> people]
               into the list "<list name>"
Emails         look up work emails for everyone found without one | none (LinkedIn only)
News           recent web news on everyone, last 12 months

THE CAMPAIGN
Channels       email first, then LinkedIn, 3 follow-ups; LinkedIn only for people without a valid email
               | LinkedIn only: an invitation, then an opener and 2 follow-ups
Accounts       LinkedIn: <label>; email: <address>
Timeline       about <2> invitations<, <n> emails> a day now (<warming up | shared with <k> other active
               campaigns>): <10> people ≈ <a week> to reach
Campaign       "<name>", English
Steps          <email day 0 → email +3 → LinkedIn invitation +2 → wait up to 14 days → message +1 → +4 → +7>
               | <LinkedIn invitation day 0 → wait up to 14 days → message +1 → +4 → +7>
Texts          Dealmaker's AI writes them from <4 | 2> templates I make from your product, offer and never-say
               lines (<first email, email follow-up, LinkedIn opener, LinkedIn follow-up | LinkedIn opener,
               LinkedIn follow-up>).
               No links in first messages.
Review mode    auto: each written message goes out when due, without your approval.
               (manual would mean you approve every message on the Queue tab; anything not approved in time is
               not sent, which can leave LinkedIn messages unsent.)
Test run       none | one test to <colleague> first, from the list "<existing test list>" | a new one-person list
               <With LinkedIn steps: if <colleague> is already your LinkedIn connection, the test skips them and
               sends nothing on LinkedIn; I include connections for the test draft only | they are not connected>
Knowledge base saved right after the campaign is created; the AI uses it to draft replies. <Placeholders
               still open: <[commission], ...>; saved without them until you give the real terms.>
               Product:    <text>
               Offer:      <text>
               FAQ:        <n> questions: <Q1>; <Q2>; ...
               Never say:  <line>; <line>; ...
Demo video     <link> (I add it to replies; links can't go into the knowledge base) | later
After launch   when you ask me: I improve the AI's reply drafts, add the demo video and a personal Booking link for
               each interested partner, check who booked, and check messages not yet sent

MONEY
Budget         a hard ceiling of <1,700> credits (~$<25.50>)
Estimate       about <1,550> credits (~$<23.25>): search up to <700>, emails up to <725> (only emails found are
               charged), news up to <125>
               I show "spent X of <1,700> credits" after every paid step.

WHAT I WILL NOT DO
I never send a message, admit people, press Launch, approve a message or a reply,
or switch AI dialogue or nudges on. Those stay yours.

WHEN I NEED YOU LATER
1. Launch: one message from me with three links: switch on AI dialogue<, and nudges> → Admit the people → Launch.
   <Before that, your to-do: <items>.> <Replace the placeholders <[commission], ...> with real terms: until then
   I save the templates and the knowledge base without those lines.>
2. Replies: when you ask "any replies?", I give you one list of reply drafts to approve and send.
   Dealmaker has no auto-send for replies.
<3. Messages: with manual review, you approve every message on the Queue tab.>

I stop and ask if a price would go over the budget or is more than 25% above this estimate, if the search would
change (more people, other filters, another place, a new list), if a check differs from this package, if an app
refuses or a result is unclear, or if anything comes up that is not in this package.

Answer "yes" to run it all, or tell me what to change.
```

### What counts as the yes

- **One explicit yes** to this message ("yes", "go", "approved", "run it"), given **after** the package was shown.
  It approves everything in the package, and nothing else.
- **"Yes, but ..."** is a change, not a yes: update the package, check the change (run its dry run or quote again if
  it touches one), and show the package again with the changed lines marked. Then wait for a new yes.
- **A yes given before the package was shown** (in the prompt, "set everything up", "ok" to the questions) does not
  count.
- **A host's approval prompt** for a tool call is a second guard, not the yes.
- "Maybe", "looks fine I guess", silence: not a yes. Ask once in one line: "Shall I run it? (yes / what to change)".
- No yes: apply nothing and do not start Prospect.

## Step C. The writes, right after the yes

In this order, one at a time, only those the package lists: **C1 suite → C2 blocklist → C3 Booking type and hours
→ C4 AI CRM.** No new question per write. But each write first runs its dry run again (or re-reads the state for a
tool with no preview), and you compare it with the package:
- **The same:** the real call with the same `args` (`confirmed` / `retryKey` if the schema lists them). Read the
  state back.
- **Already there** (a value already blocked, a meeting type the suite made that matches the package): skip it and
  say so in the report.
- **Different** (another value, another stage, a host or length the package did not show, a preview with
  `bookedOutside` the package did not mention, a refusal): stop. Say in plain words what differs and ask. Writes
  already done stay done; nothing after the stop runs until the person answers.

### C1. Suite (only if the package installs it)

`suite_install` without `dry_run`, then `suite_get` / `list_apps` until every app is ready (as in A1).
`needs_owner` or `payment_required`: give the link and stop. Then run what could not run before (A2, A3, A6, D2, D5) and compare with
the package. A template suite may add a meeting type and a partner pipeline by itself: use them when they match the
package; if they differ (another length, other stages), ask.

### C2. Blocklist (only if the package names values)

`blocklist.import` (many values) or `blocklist.add`, one call per value: `kind: "domain"` with the domain (or
`kind: "person"`), and a short `note`. After: `created: false` is "was already blocked". Report: "Blocked:
acme.com, beta.io. No campaign will reach them. You can unblock one in Dealmaker: Settings → Blocklist." Removing a
block is the person's.

### C3. Booking meeting type and hours (only if the package changes them)

- `create_type` (or `update_type` with `hosts`) without `dry_run`, after the fresh dry run matched. Report: "Booking
  now has the meeting type '<name>' at /b/<slug>. You can change or switch it off in Booking."
- `set_schedule` after its fresh dry run matched (a new `bookedOutside` the package did not show is a difference).
  Report the new hours.

### C4. AI CRM deal move (only if the package sets it)

`list_pipelines` again; the pipeline and stages must still be those in the package. Then `save_pipeline` with `id`
and `meetingStages`. Read `list_pipelines` again and confirm.

**New pipeline:** in this order, comparing each step with the package and stopping if anything differs (the
pipeline already made stays):
1. `save_pipeline` without `id` (its dry run again first), then `list_pipelines`: read the new pipeline's id and the
   stages it got.
2. Each stage the package adds or renames: `save_pipeline_stage` with that `pipelineId`, dry run, compare, real call.
3. `save_pipeline` with the new `id` and `meetingStages`, dry run, compare, real call.
4. `list_pipelines` again and confirm the stages and the deal move.

## Continuing in a new chat

The package and the budget live only in the chat where they were approved; a new chat has no yes. Read back what
was applied (`blocklist.list`, one call per `kind`; `list_types`, `get_schedule`, `list_pipelines`, `lists.list`,
`prospects.search` for the list, the campaign via `campaigns.get`, its knowledge base via `knowledge.get`). Ask, in
one message, only for what the next phases need and cannot read (Prospect needs Q5, Q7; Outreach needs Q1, Q2, Q4 if
no templates were saved, Q8 to Q11; the knowledge base needs Q1, Q2, Q4 only if it is empty; Engage needs Q3), and
for the credits left for this run. Then show a short package with only the phases still to run, their dry runs and quotes, and the
budget for them. One yes, then go on.
