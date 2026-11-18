# Book (Phase 6)

**Goal:** an interested partner books a call through a personal Booking link, and everyone can see it. Nothing here
is paid.

## 1. Before the first link

Check the meeting type with `list_types`: it is active and hosted by the person. Check the calendar with
`list_calendars`. No calendar: give the person the `connect_calendar` link; connecting it is their step.

## 2. A personal link for each interested partner

Some Dealmaker versions add the link themselves: to an untouched AI draft, or through **Insert** next to the Inbox
reply box. Keep such a line. If you edited the draft first, tell the person to use Insert.

Otherwise, make the link with `create_link`:
- the meeting type's slug;
- the partner's name;
- their `valid` email, or none;
- `ref` = their Dealmaker prospect id. `ref` is never `client_id`, which is an AI CRM agency client;
- `single_use: false`.

Dry-run it first:
- it matches the package's meeting type and host: make the real call, with no question;
- it differs ("not taking bookings", no host): stop and ask.

Each call makes a new link, valid about 30 days. Keep it in the chat and reuse it for that partner. Put it on its own
line in their reply draft; the person sends it. If the answer is lost, make a new link: the lost one just expires.

## 3. Who booked

On "any bookings?", and in every status:
1. Read Booking's `list_meetings` for dates around today.
2. It carries no `ref`, so match each meeting to a partner by the guest's email or name, and the partner meeting
   type. `get_meeting` adds the form's answers.
3. Report booked, rescheduled and cancelled meetings.

AI CRM logs a meeting on the contact, and moves the deal only if setup set that. A LinkedIn-only partner may get a
second AI CRM contact; the person merges them.

## 4. Does Dealmaker know?

When the board passes the booking on, Dealmaker marks the partner booked and stops drafting for them. Signs that it
knows:
- `inbox.get` shows `lead_booked`;
- no new AI draft for their replies;
- "Booked" in the Inbox.

If you cannot see any of these, warn the person. Dealmaker may still draft replies or nudges to the partner, and the
person should not approve them. Offer to pause that one partner in the campaign: find the tool with
`app_tools_search`, or if there is none, give the conversation's link. Act only after their yes. Never pause the whole
campaign for one partner, and never mark anyone booked yourself.

## 5. Reschedule and cancel

The guest uses the link in Booking's email. A host cancels in Booking (person only). When the person asks, draft a
short reply with a new time suggestion and the same personal link.
