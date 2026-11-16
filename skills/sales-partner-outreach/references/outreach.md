# Phase 4: Outreach

Create the draft campaign from the approved package, save its knowledge base, and hand the person their three
clicks in one message: AI dialogue, Admit, Launch. Then check the texts that are not sent yet whenever the person
asks for status. Only Dealmaker is used. No call here costs credits.

```
templates with the offer → create_draft → knowledge.save → launch hand-off (one message, three links)
→ person: AI dialogue (+ nudges) → person: Admit → person: Launch
→ Dealmaker reads recent LinkedIn posts (≤ 24 h) → AI writes the first touches → they go out in the senders' windows
   (auto) or wait for the person's approval (manual) → follow-ups are written as they come due
→ a reply anywhere stops the person
```

Words: **draft** = a saved campaign that sends nothing. **Held** = waiting for the person's admission. **Review
mode** `auto` (the package's proposal: a written text goes out when due) or `manual` (the person approves each text;
a text not approved in time is not sent, which can leave LinkedIn messages unsent).

Everything in sections 1 to 6 is covered by the package's yes: each write runs its dry run, and if it matches the
package the real call follows without a question. A difference is a stop (SKILL.md, "When you stop and ask").

## 1. Read first (free)

- `senders.list`: the LinkedIn account and mailboxes with `status: "connected"` and no `personStep`. An account the
  package named that is missing or broken: stop. "Please connect it in Dealmaker: <origin>/settings/senders. Tell me
  when it is done."
- `lists.list`: the run's list. Its `ownerMemberId` is the person's member id. Use only accounts with the same
  `ownerMemberId`: a campaign may use only its owner's accounts.
- `prospects.search` for the list (follow `cursor`): count people with a `valid` email. Count only `valid`.

Say it in one line, with numbers: "14 of 21 people have a valid email: email first, then LinkedIn. The 7 without one
get LinkedIn only." No "OK?": the package already approved the plan.

## 2. The test run (only if the package has one)

The one-person list the package named: an existing internal test list from `lists.list` when the package reuses
one, else a new list with the one colleague (`prospects.import`). Then a separate small draft with the same
templates and steps, made before the main draft. With LinkedIn steps, `excludeFirstDegree: true` (the default)
leaves out a colleague already connected to the sender, and the test sends nothing on LinkedIn. If the package says
to include connections: for a 1st-degree test person, a test draft with no invite step (message first) and
`excludeFirstDegree: false`, on the test draft only. If the dry run refuses it, or the package does not include
connections, say plainly that the LinkedIn part is not tested. There is no "email only" fallback when no mailbox is connected. A test draft from an earlier run stays as
it is: make a fresh one, unless the person said to reuse it. After making the test draft, read `prospects.search`
for the test list with the state for "excluded as 1st-degree connection" (take the exact value from the tool's
schema; never guess it; no such state: say you cannot check it) and tell the person whether the test person was
left out. The test gets a launch hand-off of its own first; the main hand-off comes when the person says the test
looked right.

## 3. People without a usable email

Read `app_tool_describe(warmleads, campaigns.create_draft)`:
- **It says an email step is skipped for a person without a usable address:** one campaign for everyone. Such a
  person skips the email steps and goes on to the LinkedIn steps (a person with only email steps left stops).
- **It does not say so** (an older Dealmaker): an email step for such a person never goes out and blocks their later
  steps. Split the list before the draft: `prospects.import` of the people without a `valid` email into a new list
  `<name> (LinkedIn only)` (rows from their records; they stay held, not duplicated), then `lists.remove` each of
  them from the first list. This carries out the package's "LinkedIn only for people without a valid email": dry
  run, then the real calls, and say what you did. Then two drafts: email + LinkedIn for the first list, LinkedIn only
  for the second (two drafts mean two Leads links and two Launch buttons in the hand-off). Do the split before the
  draft: removing someone from a campaign's list later stops them.

## 4. Templates: how the offer reaches the texts

The AI writer writes each AI step by a template. Its `instructions` and `example` reach the writer with every text.
The campaign's knowledge base is used only for replies, not for first messages.

1. `templates.list`: the workspace's templates by `kind` (`email`, `icebreaker`, `follow_up`, `invite_note`).
2. `templates.save` (no `id`: a new one) for this campaign, so other campaigns stay as they are. Dry run, then the
   real call. Only the templates the package's channels use (email + LinkedIn: all four; LinkedIn only: the last
   two):
   - `<name>: first email` (`kind: "email"`)
   - `<name>: email follow-up` (`kind: "email"`)
   - `<name>: LinkedIn opener` (`kind: "icebreaker"`)
   - `<name>: LinkedIn follow-up` (`kind: "follow_up"`)

   **The suite's starter templates** (only when this chat installed the suite): the LinkedIn opener and follow-up are
   not new ones. Change the starter `icebreaker` and `follow_up` to these texts with `templates.save` (`id` and
   `expectedVersion` from `templates.list`, the names above), as in `setup.md`, "The suite's starter items". A
   refusal because an active campaign uses one (`PERSON_ONLY`): leave it and make a new one for that step.
   `firstTouchCall`, when the schema lists it: `false` on the opener (the soft first touch: one easy question, no call
   ask). Never set it to `true` on the opener the campaign's steps use.
3. Instructions (up to 2000 characters; `example` up to 1200): who we write to, what the product is (Q1), what the
   partner gets and what we ask for (Q2), "open on one concrete detail about them", a length with margin below
   Dealmaker's limits (an AI email up to 1200 characters, an AI LinkedIn message up to 600, an invitation note up to
   180, or 280 with Sales Navigator; a longer text is not used): "under 120 words" for an email, "no more than 450
   characters, 3 sentences" for a LinkedIn opener or follow-up, the never-say lines (Q4). For follow-ups: "a short,
   new reason to talk; do not repeat the first message". Only allowed variables (`{{first_name}}`, `{{company}}`,
   ...); any other `{{...}}` is refused. **No links**: a link in a first touch is refused, and the video and booking
   links belong to replies.
4. **No placeholders.** A term still in brackets (`[commission]`, `[payout terms]`, `[deal size]`,
   `[contract length]`) never goes into a template: leave that line out and say so. When the person gives the real
   terms before Admit, update the templates (dry run first; it is what the package said).
5. Templates are final before Admit: changing one later makes texts already written wait for the person's choice,
   and a template an active campaign uses can only be changed by the person.
6. **A/B test of a call ask** (only if the package has it, and only if `templates.save` lists `firstTouchCall`): a
   separate new template `<name>: LinkedIn opener (call)` (`kind: "icebreaker"`, no `id`) with the opener's
   instructions and `firstTouchCall: true`: the first message then ends with one short question proposing a brief
   intro call (still no time, link or calendar). The soft opener stays as it is, and the campaign's steps use it. The
   experiment between the two is the person's: `<origin>/settings/templates` → Experiments → "Start experiment" on
   the campaign's opener step; no tool starts it. Put it on the launch hand-off's to-do, after the draft exists. A
   template in a running experiment is refused for changes.

`templates.save` missing: pick the existing template of the right `kind` that fits a partner pitch and say why in
the report. Then the offer reaches the first messages only through the company profile (the person edits it in
`<origin>/settings/workspace`).

## 5. Create the draft

`campaigns.create_draft`, dry run with the whole plan (steps, delays, templates), compare with the package, then the
real call. It never sends and never launches.

**The suite's starter draft** (only when this chat installed the suite): use it instead of a new draft only if the
`steps` in its `campaigns.get` answer are exactly the plan's (`setup.md`, "The suite's starter items": type, delay,
mode, template name and kind, an invitation with no note, no visit). Then
`campaigns.update` (`id`, `expectedVersion` from `campaigns.get`; `name`, `listIds`, the person's own
`linkedinSenderId` / `emailSenderIds`, `reviewMode`), dry run, compare, real call, and read `campaigns.get` for
`readiness` and `links`. Otherwise `campaigns.create_draft` as below, and tell the person the starter draft can be
deleted in the app; never delete it yourself. A test draft (section 2) is always a new draft.

Arguments (check the schema): `name` (1-150), `listIds`, `channels` (`email`, `linkedin`), `linkedinSenderId`,
`emailSenderIds` (the person's own, from section 1), `language: "en"` (always, whenever the schema lists `language`,
even as optional: the AI writes English only; leave it out only when the schema has no such field), `tone`
(`professional` default), `goal` (`conversations` default), `reviewMode` (the package), `excludeFirstDegree`, `steps` (1-12, in order).

Step fields: `type` (`email`, `invite`, `wait_accept`, `message`, `visit`), `delayDays` (0-60 after the previous
step), `mode: "ai"` with a `templateId` whose `kind` fits (email → email; message → icebreaker or follow_up; invite
note → invite_note), no `text`. An email step needs a `subject` (up to 60) even in AI mode; the AI writes each
person's own subject. `invite` without `text` goes without a note (a note is up to 180 characters, or 280 with
Sales Navigator, no pitch).
`wait_accept` with `notAcceptedDays` (1-30, default 14): not accepted in time finishes the person.

**Email + LinkedIn:**
```
email     day 0  ai  first email template, subject
email     +3     ai  email follow-up template, subject
invite    +2     no note
wait_accept      notAcceptedDays 14
message   +1     ai  LinkedIn opener
message   +4     ai  LinkedIn follow-up
message   +7     ai  LinkedIn follow-up
```
**LinkedIn only:** `invite` day 0 → `wait_accept` 14 → `message` +1 (opener) → +4 → +7 (follow-ups).

- A LinkedIn `message` reaches only people who accepted the invitation (or are already connected), so LinkedIn always
  starts with `invite` → `wait_accept`. A message before the invitation is refused. (The one exception: a test
  draft for a colleague already connected, section 2.)
- **1st-degree connections.** `excludeFirstDegree: true` (default) leaves people already connected to the sender
  out of the campaign. Set `false` only if the package said to include the person's own connections, and only when
  the tool's description says such people then skip the invitation and get the messages; an older Dealmaker
  excludes them anyway when there is an invite step. Say which applies.
- **Any reply stops the sequence** for that person on every channel, in every campaign. Dealmaker does this.

Answer: `id`, `state: "draft"`, `version`, `readiness` (what still blocks Launch, e.g. "Connect a LinkedIn sender"),
`links` (`overview`, `leads`, `queue`, `steps`). Keep `readiness` for the hand-off.

A wrong step is refused naming `steps[i]` (`-32602`): fix that step and dry-run again (a fix that changes the plan
the package showed is a stop). To change a draft's settings later: `campaigns.update` with `expectedVersion` from
`campaigns.get` (draft only; steps are changed in the app). Apart from reusing the suite's starter draft above, it is
not in the package, so only when the person asks.

## 6. Save the knowledge base (right after the draft)

The knowledge base is per campaign: what the AI may say about the product and the offer when it drafts replies.
Dealmaker refuses to switch AI dialogue on until it is saved (`KNOWLEDGE_REQUIRED`), so save it before the hand-off.
The package's yes covers it: no question.

1. `knowledge.get` (`campaignId`): `aiDialogue`, `revision`, `knowledge` {`product`, `offer`, `faq`, `neverSay`},
   `company`, `limits`, `link` (the campaign's Settings tab). `revision: 0` and empty fields: none yet.
2. Build it from the package (Setup Q1, Q2, Q4), exactly as shown there. Limits (the `limits` in the answer win):
   `product` 1500, `offer` 1000, `faq` up to 12 items (question 200, answer 600), `neverSay` up to 15 lines of 200;
   6000 in all.
   - **Too long:** shorten the field to its limit (cut whole sentences or the last FAQ items, never change the
     offer's numbers), and say in the report exactly what you cut.
   - **No placeholders:** a term still in brackets (`[commission]`, ...) is left out, never saved; the hand-off
     keeps "replace the placeholders with real terms" on the to-do, and you save the real terms when they come.
   - **No links**, email addresses, phone numbers or sensitive data: take them out. **Never send a link field
     (`loomUrl`, `bookingUrl`, any other URL field), `aiDialogue` or a booking type**, even if an older schema lists
     them.
3. `knowledge.save` with `dry_run: true` (if the tool supports it): `campaignId`, `expectedRevision` (the `revision`
   just read), and only the fields that change (`faq` and `neverSay` replace the whole list). The `changes` match the
   package (apart from cuts you reported): the same call without `dry_run`. Two campaigns: save it to each.
4. Read `knowledge.get` again: `revision` went up, the fields are there.
5. **`knowledge.save` missing:** say so, and put the content into the hand-off: "Paste this on the campaign's
   Settings tab (<links.settings>) under the knowledge base: Product: ... Offer: ... FAQ: ... Never say: ..." AI
   dialogue can be switched on only after that.

## 7. The launch hand-off: one message, three links

Read `campaigns.get` (the `version`, `links`, `readiness`) and `knowledge.get` (`company`). Then one message. Fill
the links from `links` / the `knowledge.get` `link`, else build them (`<origin>/campaigns/view/<id>/settings`,
`/leads`, `/overview`). Leave out what does not apply (no AI dialogue and no nudges: drop step 1 and renumber).

```
Your campaign "<name>" is ready, and its knowledge base is saved. Three clicks are yours, in this order:

1. Settings: switch on "AI dialogue for this campaign"<, and "Draft nudges for silent conversations">.
   <settings link>
   It works now because the knowledge base is saved.<If the tab has a "Booking meeting type" field: set it to
   "<meeting type name>".>
2. Leads: admit the people. Tick them and click "Admit selected" (or admit all).
   <leads link>
   <n> people are waiting. Admitting sends nothing by itself.
3. Launch: the "Launch" button at the top of the campaign.
   <overview link>

<Before Launch, please also: <readiness items / empty company profile / mailbox sender name, postal address, DNS>,
each with its link. Launch is refused until they are done.> <Tell me the real terms for <[commission], ...>: the
templates and the knowledge base leave them out until then.> <A/B test: <origin>/settings/templates → Experiments →
"Start experiment" on this campaign's opener step, with "<soft opener>" and "<call opener>".> <From the suite, not
used: <the starter draft "<name>" (<link>) | the invitation-note template "<name>" | the starter list "<name>">.
Nothing goes out from them unless someone launches the starter draft<; it uses your campaign's list, so please
don't>. You can delete them in Dealmaker if you don't need them.>

What happens next, on its own: Dealmaker reads each person's recent LinkedIn posts (up to a day), its AI writes the
first messages from your templates, and <they go out in your accounts' sending windows without your approval |
each one waits for your approval on the Queue tab: <queue link>>. Follow-ups are written about a day before they are
due. A reply on any channel stops that person's sequence. You can ask me "status" any time: I check the texts not
yet sent and fix small things.

I need you again only for replies. Ask me "any replies?" whenever you like: I read the Inbox, improve the AI's reply
drafts (demo video, your personal booking link) and give you one list to approve and send. Dealmaker has no
auto-send for replies, so each one needs your click.
```

You never press any of these. No tool admits people or launches; AI dialogue and nudges are person-only switches
(`PERSON_ONLY`, `-32003`). Two drafts (the split in section 3): one hand-off with the Leads and Launch links of both.

After Admit, Dealmaker reads each person's recent LinkedIn posts through the campaign's LinkedIn account (free, about
50 people a day per account; the rest are written without posts after 24 hours), then the AI writes the first
touches: the first email, the invitation note and the first LinkedIn message.

**Follow-ups.** If follow-ups stay `preparing` while the first touches are `ready`, this Dealmaker writes each
follow-up about a day before it is due, with the earlier texts in view. In manual review the person then has about a
day to approve each follow-up; nothing goes out unapproved. Tell them. If all texts of a person are written at once
(an older Dealmaker), the follow-ups were written as first messages: check that each reads as a follow-up and fix
it.

## 8. Check the texts not yet sent (on status or "texts ready?")

`campaigns.messages.list` (`campaignId`, `limit` up to 50 people; follow `cursor`; `state` filter optional).
States: `preparing` (not written yet; `waitingFor: "research"` with `researchUntil` while the LinkedIn read runs; a
held person shows `preparing` too), `ready` (written; with manual review it waits for approval), `needs_edit`
(`invalidReason`), `approved`, `sent`. Keep `id` and `revision` for edits. With `auto`, a written text goes out when
due, so check the texts not yet sent first.

For each text, read the person (`prospects.get`: `research`, `linkedinActivity`) and check:
- every fact is in their research, posts or profile; nothing invented;
- the offer and the ask match the package; nothing from the never-say lines;
- the LinkedIn opener does not repeat the email's opening; each follow-up adds something new;
- limits: an AI email up to 1200 characters, an AI LinkedIn message up to 600, an invitation note up to 180 (280
  with Sales Navigator), a subject up to 60; no placeholders or brackets; no link in a first touch; does not start
  with "I" or the person's name; English.

**Edits inside the package go ahead without asking:** removing an invented fact, a never-say line or a link, fixing
the offer to the package's words, length, placeholders, a follow-up that repeats the first message. Use
`campaigns.messages.edit` per text (`messageId`, `expectedRevision`, `text`, `subject` for an email). A final text:
no `{{variables}}`. Keep a text you edit within the limits above (a LinkedIn message up to 600 characters, not
LinkedIn's own 1900). Report each edit as before/after in the status message. A change of tone, angle or offer
that the package did not cover: propose it and wait for a yes.

An edit **never approves**. A text the person already approved is theirs (refused). A text already sent cannot be
changed. `needs_edit` with `BUSINESS_PROFILE_MISSING` on every text: the company profile is empty; give the person
`<origin>/settings/workspace`.

`needs_edit` with `invalidReason: REQUIRES_EDIT:<codes>` (e.g. `REQUIRES_EDIT:length`): the AI writer wrote the
text, but it broke a rule and is not used. Codes: `length` (over the limits above), `placeholder`, `name_start`
(starts with "I" or the person's name), `subject`, and copy or phrase checks. Fix the cause in that step's template
(`length`: ask for a shorter text, e.g. "no more than 450 characters, 3 sentences" for a LinkedIn message): before
Launch, `templates.save` yourself (dry run first); once the campaign is active, the template is the person's, so give
them the new instruction to paste. Then the person presses "Regenerate" on that text in the Queue
(<links.queue>); no tool makes the AI write again. Tell the person which texts and why.

`needs_edit` with `invalidReason: provider_error` (often every first AI message at once, right after Launch, with an
empty text; the invite step `ready`, follow-ups `preparing`): Dealmaker's AI writer could not reach its model. It is
an app or platform problem, not the person's data. An older Dealmaker also shows `provider_error` for a text that
broke a rule (most often a LinkedIn message over 600 characters): if the template does not ask for a short text,
handle it as `REQUIRES_EDIT:length` above first. Otherwise tell the person plainly: invitations still go out, but
anyone who accepts gets no first message until the text is written; ask them to report it to whoever runs their
Dealmaker or Beyonder. Do not write the text yourself with `campaigns.messages.edit` (Dealmaker's AI writer writes
these texts), and no tool makes it write again. Read `campaigns.messages.list` again on the next status.

With manual review, end with the Queue link: "<n> messages wait for your approval: <links.queue> → each person's
'Review messages' → 'Approve' or 'Approve selected'."

## 9. After Launch

Dealmaker sends by itself: each account only on its active days, in its sending window, within weekly and daily
limits that start low for new accounts (warm-up). Emails go only to a usable address, from a mailbox with a sender
name, postal address and passing DNS; Dealmaker adds the footer (name, postal address, unsubscribe link). You never
write a footer. Lowering an account's limits or turning days off: `senders.setLimits`, `senders.setSchedule` (only
lower or keep), only when the person asks. Raising them is the person's.

**Stopping.** When the person asks: `campaigns.pause` (`id`, `expectedVersion`). It may open a pause job: follow
`campaigns.jobs.get` until `ready`, then `campaigns.jobs.confirm` with its `expectedRevision`,
`expectedScanRevision` and `confirmed: true`. One account only: `senders.pause`. Resuming is the person's, in the app.

## Fixed text instead of the AI writer (only if the person asks)

Not in the package: the person supplies the text; you may suggest edits; each change waits for their yes. Fixed
steps (`mode: "fixed"`, `text`) may use `{{first_name}}`, `{{last_name}}`, `{{company}}`, `{{job_title}}`,
`{{custom.<key>}}` with `fallbacks`. A personal first line per person only when the person asks for it: one plain
sentence under 200 characters from the title, company and saved research; the person approves each; stored as
`custom.first_line` through `prospects.import` (check its `custom` limits) and used as `{{custom.first_line}}` with a
neutral fallback. The AI writer never reads custom fields.

## Done

The hand-off is sent. After the person's Launch, `campaigns.get` shows the campaign active. Progress report, then
Engage and Book when the person asks.
