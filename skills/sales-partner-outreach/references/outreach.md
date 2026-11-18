# Outreach (Phase 4)

Contents: 1. Read first · 2. Templates · 3. The draft · 4. The knowledge base · 5. The hand-off · 6. Check texts not
yet sent · 7. After Launch · Test run · Older Dealmaker versions

**Goal:** a draft campaign with its knowledge base saved, then one hand-off message with the person's three clicks.
Nothing here is paid. Every write follows the same loop: dry run, compare with the package, real call.

## 1. Read first

- `senders.list`: the package's accounts must be connected and the person's own (the same owner as the run's list).
  If not, stop: the person connects them.
- The list's people: count the `valid` emails, with numbers ("14 of 21 have a valid email; 7 get LinkedIn only").
- **First names.** A name that looks like a credential, title or junk ("Nremt-P", all caps, Dr, MBA, an emoji, a
  company name, one letter) would read "Hi Nremt-P". Show each with a proposed fix and fix only after a yes:
  re-import with the corrected `firstName`, or edit a prepared text. Never silently.

## 2. Templates

The AI writer writes each AI step from its template (the knowledge base is only for replies). Save new templates
for this campaign, only those the channels use: first email, email follow-up, LinkedIn opener, LinkedIn follow-up.
Instructions: who we write to, the product, the partner's gain and our ask, "open on one concrete professional detail
about them", the never-say lines; follow-ups: "a short, new reason to talk; do not repeat the first message".
- **Length.** Ask for "no more than 450 characters, 3 sentences" for a LinkedIn message and "under 120 words" for an
  email. Dealmaker drops an AI LinkedIn message over 600 characters (LinkedIn's own 1,900 does not apply).
- **Soft opener.** No call ask (`firstTouchCall: false`, if listed); a call ask only as a separate A/B opener the
  package has, whose experiment the person starts.
- **No links, no placeholders,** and only the variables the schema allows.
- **Final before Admit:** once active, a template is the person's. Fixed texts only when the person supplies them.

## 3. The draft

`campaigns.create_draft` with the package's name, list, channels, the person's own accounts, review mode and steps
(it never sends):
- **Email + LinkedIn:** email on day 0 → email +3 → LinkedIn invitation +2 → wait up to 14 days for acceptance →
  message +1 (opener) → +4 → +7 (follow-ups). An email step needs a subject; the AI writes each person's own.
- **LinkedIn only:** invitation on day 0 → wait 14 days → message +1 → +4 → +7.

The invitation goes **without a note**. A LinkedIn message reaches only people who accepted (or are already
connected), so LinkedIn starts with the invitation; people already connected are left out by default. A reply on any
channel stops that person everywhere. Keep `readiness` and `links` for the hand-off. A refusal naming `steps[i]`: fix
that step and dry-run again (a fix that changes the plan is a stop). `SENDER_OWNERSHIP_MISMATCH`: use only the
person's own accounts.

## 4. The knowledge base (right after the draft)

AI dialogue needs it saved (`KNOWLEDGE_REQUIRED`). `knowledge.save` the package's product, offer, FAQ and never-say
lines. The answer's `limits` win: over a limit, cut
whole sentences or the last FAQ items (never the offer's numbers) and say what you cut. No links, contact data or
placeholders, and never a link field. Read back: the revision went up. `VERSION_CONFLICT`: read again, keep what
changed, save again.

## 5. The hand-off: one message, three clicks

In this order, each with its link from `campaigns.get` `links`, or its click path:
1. **Settings:** switch on AI dialogue, and nudges if chosen.
2. **Leads:** admit the people ("Admit selected"). Admitting sends nothing by itself.
3. **Launch,** at the top of the campaign.

Add what must happen before Launch: the `readiness` items; the mailbox's sender name, postal address and DNS; an empty
company profile; the real terms for any open placeholders; the A/B experiment; unused suite items.

Then what happens on its own: Dealmaker reads recent LinkedIn posts (up to a day), its AI writes the first messages,
which go out in the sending windows (auto) or wait on the Queue (manual); follow-ups are written a day before due.
Texts are written only after Admit and Launch (`preparing` until then): nothing to review now. Say so: you check them
at Status (section 6), while not yet sent.

Close with: "I need you again only for replies. Ask me 'any replies?' whenever you like. Dealmaker has no auto-send
for replies." Never promise a schedule.

## 6. Check texts not yet sent (on status)

Read `campaigns.messages.list`. For each text not yet sent, read the person (`prospects.get`) and check:
- every fact is in their research, posts or profile, and is professional (nothing about personal life, health,
  family, politics or religion);
- the offer and the ask match the package; nothing from the never-say lines;
- no link or placeholder in a first touch; within the length limits; follow-ups add something new;
- the greeting uses a real first name (see "First names": a yes before any fix).

Fix the rest with `campaigns.messages.edit` without asking and report each fix before and after; a change of tone,
angle or offer waits for a yes. An edit never approves; an approved or sent text is the person's.

Special states (never rewrite or retry these yourself):
- **A text waits for approval, even in auto mode** (for example, one with no personal hook): list it with the Queue
  link.
- **`needs_edit` with `REQUIRES_EDIT:<codes>`** (`length`, `placeholder`, `name_start`, ...): fix that step's
  template (once active, give the person the new instruction); then the person presses Regenerate in the Queue.
- **`needs_edit` with `provider_error`:** the AI writer could not reach its model; the text is empty. Report it as an
  app problem (people who accept get no first message until it is written). Never write it yourself.
- **`BUSINESS_PROFILE_MISSING` on every text:** the person fills in the company profile.
- **An approved text stays queued for a lead in some country:** a country rule in the app may hold messages to some
  countries. Say so and point to the lead; do not retry.

## 7. After Launch

Dealmaker sends in each account's window and limits, and adds the email footer. Pause a campaign or a sender only
when asked; resuming and raising limits are the person's (`PERSON_ONLY`).

## Test run (only if the package has one)

A one-person list and its own small draft, before the main one. A colleague already connected to the sender is left
out: if the package includes connections, the test draft alone has no invitation step and includes them; otherwise
say the LinkedIn part is not tested. Tell the person whether the test person was left out. The main hand-off comes
after the test looked right.

## Older Dealmaker versions

- Pass `language: "en"` whenever the draft's schema lists it.
- `knowledge.save` may still list link fields: never send them.
- The draft tool does not say an email step is skipped without a usable email: before the draft, move those people
  to a list "<name> (LinkedIn only)" (`prospects.import`, `lists.remove`) and make two drafts.
- `provider_error` can also mean a text over its limit. If the template does not ask for a short text, treat it as
  `REQUIRES_EDIT:length` first.
