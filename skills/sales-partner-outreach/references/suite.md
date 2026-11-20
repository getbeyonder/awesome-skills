# After a suite install: the starter items

Read this only when this chat installed the suite. A template suite adds starter items in Dealmaker:
- a draft campaign with an empty list;
- message templates: a LinkedIn opener and follow-up (older sets also have an invitation note);
- a target profile.

The run reuses some of them, so the person ends up with one draft, one list and one set of templates. The package's
"suite starter items" lines cover all of this, so no new question is needed.

**Which items are the suite's.** Read Dealmaker's templates, lists and campaigns before the install (if Dealmaker was
already on the board) and again after it. Only an item that is new counts. Match by name and kind, never by an id you
remember. The platform adds the items a little after the apps are ready, so none at first is normal: read again
before Prospect. An item you cannot match counts as no starter item.

**The list.** Read its people, following the cursor:
- still empty: the search goes into it with `listId`;
- anyone in it: use the package's own new list, and say why.

**The templates.** Change the starter LinkedIn opener (`icebreaker`) and follow-up with `templates.save`, passing `id`
and `expectedVersion` from `templates.list`. Give them this run's names and texts, and name every variable the texts
use. The opener stays soft.
- `PERSON_ONLY` (an active campaign uses the template): leave it, never retry, and make a new template for that step.
- `VERSION_CONFLICT`: read again and repeat once.
- Email templates are always new. The invitation-note template is not used.

**The draft.** Read it with `campaigns.get`. Reuse it only if it is a draft and its `steps` match the plan one for
one:
- the same type, delay and mode;
- the wait's 14 days;
- the invitation with no note and no template;
- no visit step;
- each AI message step naming the template you just saved for it.

A plan with email steps never matches, and an answer without `steps` cannot be confirmed. To reuse it, call
`campaigns.update` (`id`, `expectedVersion`) with only the name, the list, the person's accounts and
`reviewMode: "manual"` (never `auto`: only the person switches a campaign to automatic).
Otherwise make the run's own draft. Tell the person the starter draft can be deleted in Dealmaker, and that it must
not be launched if it uses the same list.

**The target profile.** Never change it. It drives Dealmaker's own lead search, which this run does not use.

**Unused items.** List each one once in the launch hand-off, with "you can delete it in Dealmaker if you don't need
it". Never delete anything yourself.
