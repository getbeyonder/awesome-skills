# Setup (Phase 0)

Contents: 1. Read · 2. Ask five questions · Defaults · 3. Dry runs and the quote · 4. The package · 5. The yes and the
writes · 6. A new chat

This is one conversation before anything is searched or paid. Nothing in setup costs credits.

## 1. Read (free)

Say that you only read for now. Then check the following and report in plain words. If a check fails, say which one
and never guess its answer.

- **Company and apps.** Pass the company id from `whoami` or `companies_list`, never only the name: two companies can
  share one. Say which company you read.
  - Dealmaker, Booking and AI CRM all present: never offer the suite. It would add a second draft, list, set of
    templates, meeting type and pipeline next to the person's own.
  - Some missing: offer the catalog suite that installs all three (read it from `catalog_list`, never from memory),
    or single apps.
  - Without Dealmaker the flow cannot run.
- **Board links.** Two are needed:
  - Dealmaker → Booking (`request.Request`), for personal booking links;
  - Booking → Dealmaker (`core.Person`, `core.Event`), so Dealmaker learns who booked.

  A missing one goes into the package. Call `connect` with an explicit `things` list: a blank proposed line can make
  it fail with a conflict.
- **Sending accounts** (`senders.list`): a connected LinkedIn account and mailbox, any step the person must take, and
  each account's limits and other active campaigns. Use them for a realistic timeline (2 invitations a day while
  warming up → 10 people take about a week). A campaign may use only its owner's accounts.
- **Email lookup** (`enrichment.status`): if it is not configured, plan LinkedIn only.
- **Booking:**
  - the calendar: `connected: true` is enough, also with `scopes_short`;
  - the meeting types: one whose name or slug contains "partner" counts as found;
  - the hours and time zone: a zone of UTC is probably unset.

  If `connect_calendar` gives no link (for example the host blocked the call), give the click path. Never retry
  around a host's denial.
- **AI CRM:** the pipelines. No tool shows which pipeline a booked guest's deal lands in.
- **Company profile** (`knowledge.get`, when any campaign exists). It goes on the to-do when it is empty, describes
  another product, or has a call to action that conflicts with the offer. Dealmaker's AI writer uses it in every
  message. Add a conflicting promise to the never-say lines.

Then give a to-do with only what is missing, each item with its link or click path:
- connect LinkedIn;
- connect a mailbox with a sender name, a real postal address and passing DNS (SPF, DKIM, DMARC). US law for
  commercial email (CAN-SPAM) needs the name and address. Dealmaker adds them and an unsubscribe link to every
  email; you never write a footer;
- connect the calendar;
- fill in the company profile.

The to-do does not block the questions.

## 2. Ask five questions, in one message

Ask only what the prompt and the reads did not answer, one short line per question with no sub-questions, and your
proposal where you can make one.
Do not name tools or fields. "ok" accepts the proposals; it is not the yes to the package.

1. **Product:** what it is, who buys it, and the typical deal size. You cannot guess this one.
2. **Offer:** what a partner gets per client (a number or a clear rule), and what we ask of them (proposal: a
   15-minute intro call). A term left in brackets stays a placeholder.
3. **Partner profile:** the partner type, who in the company, where in the US, company size, and must-have title
   words. A place outside the US is out of scope: say so.
4. **Booking:** the meeting type partners book, and the person's hours. Propose the "partner" type you found, or
   creating "Partner intro" (30 min, video, two form questions: their clients, and the topic).
5. **How many** people (proposal: 25 as a test).

Ask more only when the board forces it:
- several people's accounts are visible: "which are yours?";
- an app was archived before: restore its data or start fresh is the person's choice;
- the zone is UTC, or 5 or more hours from New York: "which city are you in?", and propose a window that overlaps
  9:00–13:00 New York.

Allow at most one follow-up round. A missing partner profile blocks the package.

## Defaults

The package shows these, and the person can change any of them with a reply.

| Topic | Default |
|---|---|
| Demo video | Later. No video is promised until there is a link |
| FAQ and never-say | 2–3 Q&As drafted from the product and offer. Never say: exclusivity, a fixed income, a price for the client, anything not in the offer |
| Companies to skip | None. The person may give websites; they are blocked as domains. Never guess a domain |
| Channels | Email first, then LinkedIn, only if email lookup is on and a mailbox is connected; marked "not yet tested live". Otherwise LinkedIn only, with no email lookups in the budget |
| Accounts | The person's own connected accounts |
| Campaign name | "US partners: <type> <size> (<channels>)". Texts are in English |
| Review mode | Auto: each text goes out when due. Manual means the person approves every message on the Queue; a text not approved in time is not sent, which can leave LinkedIn messages unsent |
| AI dialogue | On for a test run. The person switches it on at Launch, and still approves every reply |
| Nudges | Off |
| Hours | Booking's current hours |
| Deal move in AI CRM | None; meetings are still logged on the contact |
| Budget | The estimate plus about 10%, as a hard ceiling in credits |
| Research | News from the last 12 months, for the whole list |
| A/B test of a call ask; test run to a colleague | None |

## 3. Dry runs and the quote (free)

Before the package, dry-run everything it will promise:
- the suite install (only if apps are missing);
- a missing board link;
- the blocklist values;
- a new Booking meeting type (never set a price or refund window);
- new hours (note meetings already booked outside them);
- an AI CRM change;
- the people-search quote (a dry run: nothing is searched or charged).

**Deal move.** Only if the person names a pipeline: set `meetingStages` on it. A new pipeline uses `standard: false`
with stages that have no required fields. Its stages and the deal move are marked "checked right after creation".

**Estimate.** Add up the search quote, emails (about 29 credits per email found) and research (about 5 credits per
person). These are typical prices; the real quote wins. The estimate must fit the budget; if it does not, propose
less.

If Dealmaker is missing, mark the lines that need it "checked right after the install".

## 4. The package

One message in plain words, with no ids or field names. Mark defaults "(default)" and leave out lines that do not
apply. It has these parts:
- **What I set up now:** each item with its dry-run result.
- **Who I find:** the profile, the count, the list, emails, and news.
- **The campaign:**
  - name, channels and steps, accounts, and timeline;
  - how texts are written: a soft first message, the invitation without a note, no links;
  - review mode, AI dialogue, nudges, and the demo video;
  - the knowledge base in full: product, offer, FAQ and never-say lines, plus any open placeholders (saved without
    them until the real terms come).
- **Suite starter items** (only with a suite install): what is reused and what stays unused.
- **Money:** the budget in credits (~$), the estimate for each paid step, and "I show 'spent X of N' after every paid
  step".
- **Yours, not mine:** the person-only steps; later, the Launch clicks (AI dialogue → Admit → Launch) and approving
  every reply; the to-do.
- **When I stop and ask.**

End with: "Answer 'yes' to run it, or tell me what to change."

## 5. The yes and the writes

Only an explicit yes given after the package counts (see SKILL.md). "Maybe" or silence: ask once, "Shall I run it?".
"Yes, but ...": make the change, check it again, and show the package again.

Right after the yes, apply the writes in this order: suite → board links → blocklist → Booking type and hours →
AI CRM. For each one, run its dry run again, then:
- same as the package: make the real call and read back;
- already there: skip it and say so;
- different: stop and ask. Writes already done stay done.

Then give a progress report and start Prospect at once.

## 6. A new chat

The package lives only in the chat where it was approved.
1. Read back what exists: the blocklist, Booking's types and hours, the pipelines, the lists and their people, the
   campaign and its knowledge base.
2. Ask only what the next phases need and cannot read, and how many credits are left for this run.
3. Show a short package for the phases still to run, with their dry runs and quotes.
4. Get one yes, then go on.

A new chat cannot tell which items an earlier suite install added, so it reuses none. Name any that are still
unused.
