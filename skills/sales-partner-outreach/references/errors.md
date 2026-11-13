# Refusals and lost answers

A refusal is an answer to explain to the person, not an obstacle to get around. After a refusal or the person's no,
do not try another way. If the Beyonder plugin's own errors skill is available, it explains the platform codes in
more detail.

**Every refusal and every `outcome_unknown` is a stop,** even inside the approved package: do what the table says
(read the state, fix an argument), tell the person in plain words, and wait before going on with anything that
depends on it. Only a `bad_request` you can fix without changing what the package said may be fixed and dry-run
again without asking.

## Platform answers (Beyonder MCP)

| Code | What it means | What you do |
|---|---|---|
| `needs_human` | Only a person can do this step | Give the link it carries (or build the screen link) and say which button |
| `needs_owner` | The company owner must act (install, allow) | Give the link to the person; they pass it to the owner |
| `payment_required` | A plan or payment is needed | Give the link; stop that step |
| `not_granted` (with `grantUrl`) | The owner has not allowed a data provider | Give the link to the owner; stop that step |
| `outcome_unknown` | The call went out and the answer was lost; it may have run (and been paid) | Read first (below). Never repeat blindly |
| `REQUEST_IN_PROGRESS` | The first run is still going | Wait and read; do not start another |
| `bad_request`, `-32602` | An argument does not fit; the answer names it (`steps[i]`, `faq[1].answer`, ...) | Fix that argument, dry-run again |
| `forbidden`, `FORBIDDEN`, `role_forbidden` | The person's role does not allow it (a viewer; a member on someone else's campaign; a non-admin in Booking or AI CRM) | Say so; the right person does it |
| `no_place` | The person has no place in the app yet | They open the app once through Beyonder |
| `module_disabled`, `feature_off` | That part of the app is off | Say so; stop that step |
| `not_found`, `NOT_FOUND` | No such record the person may see | Read the ids again; tell the person if it persists |
| `rate_limited` | Too many calls | Wait (`retryAfterMs` if given), then read |
| `unauthorized`, `app_unreachable`, `platform_unavailable`, `server_unreachable` | Sign-in or connection problem | Say so; try later; nothing to work around |

## App answers

| Answer | Where | What you do |
|---|---|---|
| `no_credits`, `NO_CREDITS` | Paid calls | Nothing was charged; stop and tell the person to top up |
| `unavailable`, `not_configured` (with `reason`) | Search, email lookup, research | Say it plainly; skip that phase (LinkedIn only for emails) |
| `nothing_to_do` | Research | Everyone cached or suppressed: read them |
| `PROSPECT_SUPPRESSED`, `skipped: suppressed` | Enrich, research | Leave the person out |
| `ENRICHMENT_ALREADY_PENDING` | Enrich | Do not repeat; read later |
| `PROSPECT_NOT_FOUND` | Research | Read the ids again; the whole call was refused |
| `IDEMPOTENCY_ARGUMENT_MISMATCH` | Any keyed write | Never change `args` on a retry |
| `-32003`, `PERSON_ONLY` | Dealmaker: a person-only step (approved text, running campaign, AI dialogue, nudges, resume, raising a limit) | Hand it to the person with the link |
| `KNOWLEDGE_REQUIRED` | Switching AI dialogue on (the person's) | The campaign's knowledge base is not saved: save it (`outreach.md`, section 6), then the person tries again |
| `VERSION_CONFLICT` | `campaigns.update`, `campaigns.messages.edit`, `knowledge.save`, `inbox.drafts.edit`, `templates.save` | Read again, keep what changed, then edit |
| `DRAFT_STALE` | `inbox.drafts.edit` | The partner wrote again: read the conversation, edit the new draft |
| `SENDER_OWNERSHIP_MISMATCH` | `create_draft`, `campaigns.update` | Use only accounts with the person's `ownerMemberId` |
| `readiness` not empty | After `create_draft` | Tell the person what to fix, with the link |
| `needs_edit` + `BUSINESS_PROFILE_MISSING` | Messages list | The person fills the company profile |
| `CAMPAIGN_NOT_READY` | Launch (the person's) | Fix `readiness` first |

## After `outcome_unknown`: read first

| The lost call | Read this | Repeat only if |
|---|---|---|
| `suite_install`, `app_install` | `suite_get`, `list_apps` | the apps are not there or not installing |
| `people.search` with a list | `lists.list`, then `prospects.search` for that list | the list has no people from this search |
| `prospects.import`, `lists.remove` | `prospects.search` for the list | the change is not there |
| `prospects.enrich` | after a few minutes `prospects.get` (`prospects.enrichStatus` only if `apps_tools_list` lists it) | no job and no email |
| `prospects.research` | after a few minutes `prospects.get` for a few people | nobody has new `research` |
| `blocklist.add` | `blocklist.list` with that value's `kind` | the value is not blocked |
| `templates.save` | `templates.list` | the template is not there |
| `campaigns.create_draft` | `campaigns.list` | the campaign is not there |
| `campaigns.messages.edit` | `campaigns.messages.list` | the text did not change |
| `knowledge.save` | `knowledge.get` | `revision` did not change |
| `inbox.drafts.edit` | `inbox.drafts.list` with `threadId` | the draft did not change |
| Booking `create_type`, `set_schedule` | `list_types`, `get_schedule` | the change is not there |
| Booking `create_link` | nothing to read | make a new link; the lost one only expires |
| AI CRM `save_pipeline` | `list_pipelines` | `meetingStages` did not change, or the new pipeline is not there |
| AI CRM `save_pipeline_stage` | `list_pipelines` | the stage is not there |

When you repeat, use the same `args` and the `retry_token` from the `outcome_unknown` answer. Under the same key the
app runs (and charges) at most once.
