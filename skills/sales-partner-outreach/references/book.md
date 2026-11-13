# Phase 6: Book

The partner books a call on Booking's page through a personal link. Booking keeps the meeting, AI CRM logs it on the
partner's record (and moves the deal, if Setup set that), and Dealmaker should stop drafting for that partner. No
call here costs credits.

```
partner wants a call → a personal Booking link in the reply (ref = the partner's Dealmaker id) → the person sends it
→ partner books on Booking's page → Booking: meeting booked
   → AI CRM: a "Meeting" activity on the partner's contact (+ the deal moves, if set)
   → Dealmaker: "Booked": AI drafts and nudges stop for them (only when the board passes the booking on)
```

**Person only:** connect their Google calendar (Booking `connect_calendar` gives the link; choosing calendars is
`set_calendars`, a person-only tool); set the campaign's booking meeting type in Dealmaker, when that setting exists;
send the reply with the link; cancel a meeting as the host (Booking `cancel` is person-only). Booking's writes have
their own preview: `dry_run: true` on `app_write` comes back as Booking's preview in `appPreview`.

## 1. One-time setup (done in Setup; repeat only what Setup could not finish)

- **Meeting type:** `list_types` (`slug`, `active`, `bookingUrl`, `hostMembers`). Missing: `create_type` (Setup
  C3). Wrong or no host: `update_type` with `hosts`. Never set `price` or a refund window. Setup did not finish it
  (a stop, a refusal): it is still covered by the package only if the values match it; otherwise ask.
- **Hours:** `get_schedule`; `set_schedule` only if the package (or the person, later) changes them (Setup C3).
- **Calendar:** `list_calendars` (`connected: true` is enough); else the `connect_calendar` link for the person (no
  link back, e.g. the host blocked the call: the click path to Booking's calendar settings, as in `setup.md` A4;
  never retry around the denial). Check again before the first partner books.
- **Deal move:** AI CRM `list_pipelines` / `save_pipeline` `meetingStages` (Setup C4).
- **Board links:** Dealmaker → Booking and Booking → Dealmaker, read in `list_apps` (Setup A1, C1b).

## 2. The personal link per partner

A personal link opens the meeting type's page with the partner's name and email filled in, and carries `ref`, an id
we choose: the partner's Dealmaker prospect id. Booking keeps `ref` on the meeting and sends it with the booking, so
the board can tell which partner booked even if they type another email.

**When Dealmaker adds the link itself.** Some Dealmaker versions ask Booking for the link and add it as the last
line of an untouched AI draft, or show "Personal booking link from Booking" with **Insert** next to the Inbox
composer when the draft was edited first. This needs the campaign's booking meeting type (the person sets it on the
campaign's Settings tab) and the board link Dealmaker → Booking (`request.Request`) that carries the request for the
personal link; the booked signal comes back on the link Booking → Dealmaker (`core.Person`, `core.Event`). If the
draft already ends with such a link, keep that line when you edit. If you edit before it arrives, tell the person to use **Insert**.

**Otherwise (the fallback): make the link by hand.** For each interested partner whose draft has no Booking link:
1. `create_link` with `dry_run: true`: `type` (the meeting type's slug), `name`, `email` (their `valid` one, else
   leave it out), `ref` = the partner's `prospect.id` from `inbox.drafts.list`, `single_use: false` (the link stays
   good after a cancel and can be reused in every conversation with them). `ref` is never `client_id` (that is an
   agency's client). Any Booking member may.
2. Compare the preview with the package (the meeting type, its length and host). It matches: the real call, no
   question (the package covers personal links for interested partners; a link is sent to nobody by itself). It
   differs (another type, no host, "not taking bookings"): stop and ask.
3. The real call answers `url`. Each call makes a new link: keep it in the chat and reuse it for that partner (it
   lives about 30 days).
4. Put it into the draft with `inbox.drafts.edit` (Engage step 5), on its own line after the ask. Remove any other
   call link. The person sends it.

A refused `ref`: use the bare prospect id (1-128 of letters, digits, `:`, `_`, `-`). "Not taking bookings" or "no
meeting type": `list_types`, fix the slug, or ask the person to switch the type on.

## 3. The partner books (no call)

The partner picks a time (hours minus calendar busy time) and fills the form. Booking saves the meeting, writes it to
the host's calendar when connected, emails both sides with a reschedule / cancel link, and tells the board.
- AI CRM finds the contact by the guest's email or phone (or makes a new one) and adds one "Meeting" activity; a
  reschedule or cancel updates the same activity. It moves the deal only if `meetingStages` is set. A LinkedIn-only
  partner has no email in Dealmaker, so AI CRM may make a second contact: tell the person, they merge them.
- Dealmaker, **when the board passes the booking on**, marks the partner booked: their sequences stop, untouched AI
  drafts are discarded, no new AI drafts or nudges are written, and the Inbox shows a "Booked" badge. The person and
  you can still write replies. A cancellation lets the AI draft again; stopped sequences stay stopped.

## 4. "Any bookings?": who booked

No tool tells you someone booked; read when the person asks, and in every status or "any replies?" check. These
reads need no question. Say once: "Ask me 'who booked?' whenever you like."

1. Booking `list_meetings` (`from`, `to` as dates around today, at most 400 days apart; `status` optional:
   `booked`, `confirmed`, `held`, `no_show`, `cancelled`, `pending_payment`). A member sees their own meetings; an
   admin all.
2. The list has no `ref`. Match each meeting to a partner by the guest's email or name and the partner meeting type.
   `get_meeting` (`meeting`) adds the form's answers and the meeting's history.
3. Optional: AI CRM `list_activities` for the partner's contact (find the contact id first with the AI CRM search
   tool; describe it before the call).
4. Report: "2 partners booked: <name> (<day, time, zone>) and <name>. 1 meeting was cancelled: <name>."

## 5. Does Dealmaker know about the booking?

No Dealmaker tool shows the booked flag directly. Signs that it knows: `inbox.get` shows `processingError:
"lead_booked"` on a message after the booking, `inbox.drafts.list` has no AI draft for the partner's new replies,
or the person sees "Booked" in the Inbox. Some versions also show the link's state in `prospects.get`
(`bookingLink`).

**If you cannot see that it knows,** assume it does not, warn the person, and offer to pause that partner in the
campaign: "<name> booked a call, but Dealmaker does not seem to know. Their campaign steps already stopped at their
reply, but Dealmaker may still draft AI replies and, if nudges are on, nudges for them. Please do not approve those;
answer them yourself in the Inbox. Shall I pause <name> in the campaign so nothing more is drafted for them? (yes /
no)"
- **Act only after their yes.** This is outside the package.
- **How to pause one person:** look with `app_tools_search` (Dealmaker) for a tool that pauses or stops one person
  in a campaign; `app_tool_describe` it, dry run, then the real call. No such tool: say so and give the person the
  conversation link (`<origin>/inbox/<threadId>`) or the campaign's Leads tab to do it there. Never pause the whole
  campaign for one partner, and never use `lists.remove` for this unless the person asks for exactly that.
- Never mark a partner booked yourself and never change the board's links for this unless the person asks or the
  approved package lists the link; then use the board tools with a dry run first (and the person's yes, when the
  package does not list it).

## 6. Reschedule and cancel

The guest reschedules or cancels with the link in Booking's email; the host cancels in Booking (person only). When
the person asks, read the conversation (`inbox.get`) and offer a short reply with a new time suggestion and the same
personal link.

## Done

The partner's meeting is in Booking and on their AI CRM record. The team takes the call from there.

| Call | Kind | Key args | Key answer |
|---|---|---|---|
| Booking `list_types` | Read | none | `slug`, `active`, `bookingUrl`, `hostMembers` |
| Booking `list_calendars` / `connect_calendar` | Read | none / `member?` | `connected`; `available`, `link` |
| Booking `create_link` (dry run first; no question when it matches the package) | Write | `type`, `name`, `email`, `ref`, `single_use` | `url` |
| Dealmaker `inbox.drafts.edit` | Write | as in Engage | the draft with the link |
| Booking `list_meetings` | Read | `from`, `to`, `status` | `type`, `guest`, `host`, `atIso`, `status` |
| Booking `get_meeting` | Read | `meeting` | answers, history |
| AI CRM `list_activities` | Read | `entity: "contact"`, `id` | the "Meeting" activity |
| Dealmaker `inbox.get` | Read | `id` | `processingError: "lead_booked"` |
