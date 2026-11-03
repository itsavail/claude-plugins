---
name: availnet-agency
description: Guides use of the Availnet Agency connector for a talent agency. Use when the user asks about their agency's creators in Availnet, adding a creator profile by handle or URL, editing a creator's profile, products, rates or marketplace listings, creator discovery and search, audience or performance data, brand safety or past brand work, rosters and roster share links, upcoming integration or monthly DBS inventory and availability, recurring inventory rules, or Campaign Reports.
---

# Availnet Agency

The Availnet Agency connector reads one talent agency's data in Availnet. It can also look up a public creator and add a creator profile, edit a creator's profile, products, rates and marketplace listings, create a roster, refresh a roster's public share snapshot, create an empty inventory slot, change a slot's date, title, block or rate, add a brand claim, change a claim's role or remove it, create a recurring inventory rule, update that rule's brand and rate, and create, edit, refresh and reopen Campaign Reports. It acts as the signed-in user, for the one agency they chose when connecting, with only the permissions they allowed.

## Definitions / glossary

- **Creator**: a person the agency represents. Tools identify the creator's record by `creator_id`. One creator can have several social accounts, including more than one on a platform.
- **Channel** (Creator Social Account): one creator account on one social platform, identified by `social_account_id`. It is separate from the creator's Availnet login.
- **Connected channel**: a channel the creator authorized for Availnet through its platform. **Connected creator**: a creator with at least one connected channel, the same rule as the Connected tab on Creator Profiles in Availnet. Connection is per channel. One connected channel says nothing about the creator's other channels. Agency or roster membership, a public profile or a stored handle doesn't make a creator connected.
- **Reauthorization required**: a connected channel whose authorization expired, was revoked or lacks a permission. It still counts as connected, but its stats, inventory or reports may be unavailable until the creator reconnects it. Availnet shows it on the channel as Reconnect required or Limited access. **Not connected**: any other channel, such as one the creator never authorized.
- **Public creator result**: a creator returned by `agency_list_creators` or `agency_search_creators`. Being returned means discoverable, not connected.
- **MCP connection**: the user's authorization of this connector for one agency, with the permissions they allowed. It is separate from a creator authorizing a channel.

List and search results carry no connection status and take no connected filter. Don't call returned creators connected, invent a `connected` filter or give a count of connected creators. Say the connector can't tell, and point the user to the Connected tab on Creator Profiles. A tool's account-level reason describes that account for that tool only. Inventory `not_connected` and report `channel_disconnected` also cover channels the Connected tab counts as connected, such as one missing a permission, so don't report either as Not connected. Don't extend any reason to the creator's other channels. Unsupported Instagram inventory, a missing report account or a null count isn't evidence that a channel is not connected.

## Start with context

Call `agency_get_context` before any other tool. It returns:

- `organization`: the agency you act for. Name it in answers so the user knows where the data came from.
- `role`: admin, manager, member or viewer.
- `capabilities[]`: `read`, `reports_read`, `reports_write`, `reports_create`, `reports_update`, `rosters_write`, `availability_write`, `bookings_write`, `creators_write` and `creators_update`, each with `allowed` and, when not allowed, a `reason`.

When a capability is not allowed, don't call tools that need it. Explain instead:

- `reason: missing_scope`: the user can fix it. Tell them to reconnect the Availnet Agency connector and allow that permission on the Availnet screen.
- `reason: role_denied`: reconnecting won't help. Their role in the agency doesn't allow it, and an agency admin must change the role.

One connection serves one agency. To work in another agency, the user reconnects and chooses it.

## Which tool answers which question

| The user wants | Tool |
|---|---|
| The agency, role and permissions in effect | `agency_get_context` |
| The agency's creators, biggest first, with no filter | `agency_list_creators` |
| Creators matching a topic, platform, audience or size | `agency_search_creators` |
| One creator's detail, audience, brand safety, brand work or availability | `agency_get_creator` |
| Look up a public creator by handle or URL before adding them | `agency_lookup_creator_profile` |
| Add that looked-up creator, or decline them (a change; confirm first) | `agency_confirm_creator_profile` |
| A link a creator opens to join and connect their channels (a change; confirm first) | `agency_get_creator_invite_link` |
| A creator's products and rates, marketplace listings or content profile ids | `agency_get_creator` |
| Edit a creator's name, bio, categories, keywords, brand work, products and rates, CPMs, units per month, marketplace listings or YouTube view sharing (a change; confirm first) | `agency_update_creator_profile` |
| The agency's rosters | `agency_list_rosters` |
| One roster and its creators | `agency_get_roster` |
| A roster's public share link | `agency_get_roster_share_link` |
| A new roster from selected creators (a change; confirm first) | `agency_create_roster` |
| Republish what buyers see at a roster's link (a change; confirm first) | `agency_refresh_roster_snapshot` |
| Monthly inventory counts for a creator over any range | `agency_get_creator_availability` |
| Individual upcoming inventory slots | `agency_list_inventory_slots` |
| One slot's status, claims and revision | `agency_get_inventory_slot` |
| An empty slot on a connected YouTube account (a change; confirm first) | `agency_create_inventory_slot` |
| A brand claim on a slot (a change; confirm first) | `agency_add_inventory_claim` |
| A claim's role on a slot (a change; confirm first) | `agency_set_inventory_claim_role` |
| Remove a claim from a slot (a change; confirm first) | `agency_remove_inventory_claim` |
| A slot's date, title, block or rate (a change; confirm first) | `agency_update_inventory_slot` |
| Recurring inventory rules for one account | `agency_list_inventory_rules` |
| A new recurring inventory rule (a change; confirm first) | `agency_create_inventory_rule` |
| Change a recurring rule's brand and rate (a change; confirm first) | `agency_update_inventory_rule` |
| Create one YouTube or Instagram Campaign Report (a change; confirm first) | `agency_create_report` |
| Edit a Campaign Report (a change; confirm first) | `agency_update_report` |
| Refresh a live Campaign Report's numbers (a change; confirm first) | `agency_refresh_report` |
| Reopen an expired Campaign Report (a change; confirm the expiration first) | `agency_reopen_report` |
| The agency's Campaign Reports | `agency_list_reports` |
| One Campaign Report's analytics | `agency_get_report` |
| Supported report settings | `agency_get_report_configuration` |
| Eligible report accounts | `agency_list_report_accounts` |
| Media for one report account | `agency_list_report_media` |
| A Campaign Report's public share link | `agency_get_report_share_link` |

## Creators

`agency_list_creators` pages through all of this agency's public YouTube and Instagram creators, ordered by followers. It takes only `include`, `limit` and `cursor`. To narrow the list, use search.

`agency_search_creators` finds public YouTube and Instagram creators in this agency.

- `ranking` is `structured` (default) or `semantic`.
  - Structured matches literal text and orders by followers. Use it for names, handles and pure filters.
  - Semantic runs the agency's smart search on `query` and orders by similarity. It needs a nonblank `query`. Use it for descriptive asks such as "cozy home cooking creators".
  - Semantic search has its own small per-minute budget, and every page reruns the search. Don't page through semantic results without a reason.
  - A response with `ranking: "structured"` to a semantic request means the query had no semantic intent.
- `filters` takes `platforms` (youtube, instagram), `categories` (matches any), `followers_min`, `followers_max`, `excluded_creator_ids` and `conditions`. All filters AND together.
- Each `conditions` entry is `{ field, operator, value, params }`:
  - `followers_count`, `average_views`: `>`, `>=`, `<`, `<=` or `=` with an integer.
  - `engagement_rate`: the same operators with a nonnegative number.
  - `audience_country_pct` with `params.country` (two-letter code), `audience_gender_pct` with `params.gender` (male, female, other, unknown), `audience_age_pct` with `params.age_group` (age13-17, age18-24, age25-34, age35-44, age45-54, age55-64, age65-). Values are percentages from 0 to 100: 30 means 30%, and 0.5 means 0.5%.
  - `platform`: `=` with youtube or instagram, or `in` with an array of them.
  - `creator_name`: `=` or `contains` with text.
- Unknown keys, fields or operators fail with `validation_failed`. Fix the arguments; don't retry them unchanged.
- Each result carries `match_reasons`. Use them to explain why a creator matched.

`include` adds sections and defaults to none. Request only what the question needs: `bio`, `performance`, `audience`, `brand_safety`, `brand_work`, or `full` for all of them. `full` is heavy on lists, so name sections there. `availability` works only on `agency_get_creator`; lists, searches and rosters reject it. On `agency_get_creator`, `full` also adds `availability`, four months of inventory counts per account.

Every requested section comes back in an envelope:

- `state: "available"` with `data`.
- `state: "empty"`: Availnet has nothing stored. Say so.
- `state: "unsupported"`: the stored data couldn't be read. Say it's unavailable.

A null metric means unknown, never zero.

`agency_lookup_creator_profile` is how you add someone who is not already in the agency. `agency_search_creators` only finds people already in the agency.

- Send a YouTube or Instagram `url`, or a `handle` with `platform` (`youtube` or `instagram`). Do not send both.
- Lookup has a small per-minute budget. On `rate_limited`, wait the stated seconds. `service_unavailable` means lookup is paused; retry shortly.
- Show the user the display name and each channel: platform, handle, followers, and whether it is already in this agency. Ask whether to add the profile. People usually add every channel, so do not ask which ones to drop.
- A channel with `already_in_agency: true` carries the existing `creator_id`. When every channel is already in the agency, the creator is already here: say so and read them with `agency_get_creator` instead of offering to add them.
- If `creators_write` is not allowed with `missing_scope`, tell them to reconnect the Availnet Agency connector and allow Add and edit creator profiles. Do not call lookup or confirm.
- Pass `confirmation_token` through unchanged to `agency_confirm_creator_profile`. Never rebuild channel objects. The token lasts until `expires_at`. A `validation_failed` for an invalid or expired token means look the creator up again and reconfirm with the user.
- `removed_channel_ids` stays empty unless they say they want to leave some channels out. Then pass those `channel_id`s, each once. An id not in the lookup is rejected. Empty keeps every channel that is not already in the agency. Channels already in the agency are skipped even if you leave them in, including a channel added after lookup.
- If they say no, call confirm with `decision: "reject"`. That creates nothing.
- Lookup and confirm need `creators_write`. Confirm needs an `idempotency_key` for both accept and reject. Always confirm with the user before calling accept. Optional `first_name` and `last_name` override the looked-up name. Retry the same key and payload to recover a lost response, including after the confirmation token expires. A different payload with that key is rejected.
- On `created: true`, `creator_id` is the new profile. A `conflict` means no YouTube or Instagram channel was left to add.
- Availnet syncs a new profile's metrics and audience, and for YouTube its brand safety and brand work, in the background. `agency_get_creator` sections can be empty until that finishes; say so rather than reporting no data.
- A new profile has no connected channels. If the user wants the creator to connect, offer `agency_get_creator_invite_link` with the new `creator_id`.

`agency_get_creator_invite_link` returns the invite-to-connect link for one creator in the agency, the same link the agency portal's invite button copies. The creator opens it to join Availnet and connect their channels.

- Pass the creator's `creator_id`. It needs `creators_write`; no idempotency key. Always confirm the creator with the user before calling it, since it creates the invite and marks the creator's channels as invited when none is pending.
- It reuses the creator's pending invite (`is_existing: true`) or creates one and marks the creator's channels as invited. Calling it again returns the same link.
- It sends no email. Give the user `invite_url` to share with the creator. Anyone with the link can join as that creator, so tell them to send it only to the creator.
- A `conflict` means the creator already joined Availnet and needs no invite.

`agency_update_creator_profile` edits what the creator profile page in the agency portal can edit. It needs `creators_update` and an `idempotency_key`. Always confirm the exact change with the user before calling it. The edit is live at once, in the portal and in what marketplaces read.

- Read first with `agency_get_creator`. It returns `products` (stored rates and `included_items`), `partner_marketplaces` and `profile_targets` (each with its full `recent_brand_work`).
- Send only the fields to change. Omitted fields stay as they are. The last save wins, including over edits someone makes in the portal.
- `stage_name` is the creator's name. `profile` edits one content profile: pass its `profile_id` from `profile_targets` with `biography`, `categories` (1 to 3), `keywords` (up to 3) or `recent_brand_work` (up to 10, replacing the list: to add a brand, send that target's `recent_brand_work` back with the new entry; if it is `null`, edit brand work in the portal). A `sponsorship_date` needs a time zone offset, such as `2026-05-01T00:00:00Z`.
- `products` replaces the whole list in the order sent. Keep a product by sending its `id`, and send its `included_items` again: an omitted `included_items` saves as none. `[]` deletes every product and every marketplace listing. Prices are USD decimal strings such as `"1500"`.
- `partner_listings` lists or unlists the creator per marketplace by `partner_grant_id`. Marketplaces you leave out stay as they are. Listing needs products. If `partner_marketplaces.enabled` is false, don't offer listing.
- A `conflict` on listing means nothing in the request was saved. Follow its detail: the creator may have no products, the marketplace may no longer be enabled, or the agency keeps this creator's rates internal. Only for internal rates, tell the user to change pricing on the creator's profile in the agency portal, since the connector can't change that setting. Unlisting always works.
- `account` edits one channel by `social_account_id`: `integration_cpm`, `dbs_cpm` and `units_per_month_override` (`null` returns to the computed estimate).
- `channel_metrics_sharing` shows or hides YouTube views on the creator's public shares.
- Profile images and pricing visibility can't be changed here; point the user to the portal.
- Everything in one call saves together or not at all. `search_index_refresh: "retryable_failure"` means the edit saved but search re-indexing wasn't queued: retry the same key and payload. The same key and payload replay without saving twice while the replay is retained (24 hours by default); after that, the same key saves again and can overwrite newer edits.

## Which ID goes where

| ID | Comes from | Pass it to |
|---|---|---|
| creator `creator_id` | list `creators[]`, search `items[]`, roster `members[]`, lookup `channels[]` when `already_in_agency` is true, confirm when `created` is true | `agency_get_creator`, `agency_get_creator_availability`, `agency_list_inventory_slots`, `agency_get_creator_invite_link`, `agency_update_creator_profile`, search `excluded_creator_ids` |
| `confirmation_token` | `agency_lookup_creator_profile` | `agency_confirm_creator_profile`, unchanged |
| `channel_id` | lookup `channels[]` | `removed_channel_ids` only when the user wants that channel left out |
| `social_account_id` | `social_accounts[]` on any creator | `agency_list_inventory_slots` as `creator_social_account_id`, `agency_update_creator_profile` `account` |
| `profile_id` | `agency_get_creator` `profile_targets[]` | `agency_update_creator_profile` `profile` |
| `partner_grant_id` | `agency_get_creator` `partner_marketplaces.listings[]` | `agency_update_creator_profile` `partner_listings` |
| product `id` | `agency_get_creator` `products[]` | `agency_update_creator_profile` `products`, to keep that product |
| `roster_id` | `agency_list_rosters`, `agency_create_roster` | `agency_get_roster`, `agency_get_roster_share_link`, `agency_refresh_roster_snapshot` |
| slot `id` and `kind` | `agency_list_inventory_slots` | `agency_get_inventory_slot`, `agency_add_inventory_claim`, `agency_update_inventory_slot` as `slot_id` and `kind` |
| report `id` | `agency_list_reports` | `agency_get_report` and `agency_get_report_share_link` as `report_id` |

To read a creator's inventory, list or search creators and copy the creator's `creator_id`:

- For monthly counts over any range, call `agency_get_creator_availability` with that `creator_id`. It covers every social account of the creator. `agency_get_creator` with `include: ["availability"]` covers only the creator's public social accounts, for the current UTC month and the next three; use `agency_get_creator_availability` for every account or another range.
- For a creator's individual slots, call `agency_list_inventory_slots` with that `creator_id`. It lists slots across all the creator's accounts.
- For one account's slots, copy its `social_accounts[].social_account_id` into `creator_social_account_id`. `social_account_id` is an equivalent name for the same argument; send only one of them.

Creator Profile IDs are not accepted; a creator ID that isn't this agency's answers `not_found`.

## Rosters

- `agency_list_rosters` returns each roster's `creator_count`, `public_reach` and `share_ready`.
- `agency_get_roster` returns the roster and a page of its public creators. It takes the same `include` sections as search, without availability.
- `agency_get_roster_share_link` returns `share_url: null` when `share_ready` is false: the roster has no published snapshot yet. Never build a share URL yourself.
- `agency_create_roster` creates one roster from 1 to 100 creator ids and stores its share snapshot in the same step. Success means the snapshot is stored. It needs an idempotency key and `rosters_write`. Always confirm with the user before calling it. The link comes from `agency_get_roster_share_link`. It sends no email and does not edit an existing roster. Retrying the same key and payload returns the original roster. A different payload with that key is rejected.
- `agency_refresh_roster_snapshot` republishes the public snapshot from the roster's current creators, which changes what buyers see at the share link. It creates the link if none existed. It needs `rosters_write`. Always confirm with the user before calling it. It doesn't change the title, membership or visibility, and it sends no email.

## Upcoming inventory

- Kinds are `integration` and `monthly_dbs`. They are different products: report them separately and never add them together.
- Months are `YYYY-MM` UTC calendar months. `from_month` to `to_month` is inclusive, at most 12 months per call.
- Each row has counts for `available`, `interest`, `on_hold`, `booked`, `unavailable` and `completed`.
- `available_inventory_count` is available plus interest. Call it "available, including interest", or show interest separately. Don't add interest a second time.
- An account with `supported: false` has a `reason` (such as `not_connected` or `reauthorization_required`) and null counts. Null means unknown, never zero. Zero counts on a supported account mean it has no slots.
- Quote `as_of` when you report inventory. Reads are live, not a snapshot.
- `agency_list_inventory_slots` needs `kind`, exactly one of `creator_id`, `creator_social_account_id` or `social_account_id`, and the month range. Continue `next_cursor` with the same selector and filters. `statuses` optionally filters to available, interest, on_hold, booked, unavailable or completed.
- `agency_create_inventory_slot` creates one empty unclaimed slot for an eligible connected YouTube account. It needs `availability_write` and an idempotency key. Always confirm with the user before calling it.
- `agency_list_inventory_rules` needs `social_account_id` and an inclusive `from_month`/`to_month` of at most 12 months. It returns rules whose window overlaps that range, including a rule that started earlier and is still running. Integration only. Monthly DBS has no rules.
- `agency_create_inventory_rule` creates one weekly or monthly rule for a `social_account_id` (the Creator Social Account that receives it) and applies it immediately to that account's integration upcoming slots. It needs `availability_write` and an idempotency key. Always confirm with the user before calling it. A brand name also needs `bookings_write` and books those dates. Send either `price_override` or `cpm_override`, never both. Retrying the same key and payload returns the original rule and summary. A different payload with that key is rejected.
- `agency_update_inventory_rule` changes the brand and the price or CPM on a rule from `agency_list_inventory_rules`. Cadence, start, and end stay fixed. It needs `availability_write`, `bookings_write`, and an idempotency key. Always confirm with the user before calling it. Send `brand_name` and `brand_domain` (null clears the brand) and both `price_override` and `cpm_override`. Set one number and send the other as null. Omitting either key is rejected, and both numbers together is rejected. Completed and unavailable slots keep their current brand and rate. Retrying the same key and payload returns the original rule and counts. A different payload with that key is rejected. It does not delete a rule.
- `agency_add_inventory_claim` adds one brand claim (name, optional domain, role `interest`, `on_hold` or `booked`) to a slot. It is live at once, like the Agency console, and the slot status becomes its highest claim role. Pass the slot's `revision` from your latest read as `expected_revision`. A `conflict` may mean a stale revision, a blocked or completed slot, or an unsupported account: follow its `detail`, and only for a stale revision read the slot again and confirm with the user before retrying. It needs `bookings_write` and an idempotency key. Always confirm the brand, role and slot with the user before calling it.
- `agency_set_inventory_claim_role` changes one existing claim's role and `agency_remove_inventory_claim` removes one claim. Pass the `claim_id` from the slot's `claims` and the slot's `revision` as `expected_revision`. Both are live at once, like the Agency console, and recompute the slot status from the remaining claims (available when none remain). Conflicts work as for adding a claim. A `not_found` claim was removed or belongs to another slot: read the slot again. Both need `bookings_write` and an idempotency key. Always confirm the brand, the change and the slot with the user before calling them.
- `agency_update_inventory_slot` changes the date, title, block, price override or CPM override of a slot and keeps its claims. It is live at once, like the Agency console. Send only the fields to change: `scheduled_date` `YYYY-MM-DD` moves the slot to that date's month, `scheduled_date: null` clears the date and keeps the month, `title: null` clears the title, `blocked: true` blocks the slot and keeps its claims, `blocked: false` unblocks it and restores its status from its claims. `price_override` and `cpm_override` are numbers with at most 2 decimal places, or null: omit one to leave it, null clears that one, and a number sets it and clears the other. Both numbers together is rejected. A blocked slot takes a date, title or rate change only with `blocked: false` in the same call. Pass the slot's `revision` from your latest read as `expected_revision`. A `conflict` may mean a stale revision, a completed or blocked slot, or an unsupported account: follow its `detail`, and only for a stale revision read the slot again and confirm with the user before retrying. It needs `availability_write` and an idempotency key; a slot with any active claim, interest included, also needs `bookings_write`. Always confirm the slot and the change with the user before calling it.
- A slot's `allowed_actions` names changes the user may be eligible for. A completed slot has none, and a blocked slot offers only `agency_update_inventory_slot`. This connector can create an empty slot and a recurring inventory rule, update that rule's brand and rate, change a slot's date, title, block, price override or CPM override, and add, change the role of or remove a claim; applying a rule may claim, promote, reprice or reuse matching slots. It cannot move a slot to another month without a date, delete a slot or rule, or rename a claim.
- Slot reads return `price_override` and `cpm_override` as decimal strings or null. Notes, completed price, channel CPM and video assignments stay omitted. A recurring rule returns its own `price_override` or `cpm_override` as numbers.

## Campaign Reports

- `agency_create_report` needs `reports_create`, explicit `agency:read`, `agency:reports:read` and `agency:reports:write` consent, and live Analytics create permission. Confirm the YouTube videos (1–50) or Instagram reels/collected stories from one account, settings and public note with the user before creating. The report and its note are publicly addressable immediately; there is no private draft, publish step or email.
- Send `platform: "youtube"`, `social_account_id`, `video_id` as one id or an array of 1–50 ids, `title`, supported `metrics`/`dimensions`, and a mandatory `idempotency_key`. Never send `video_ids` or channel/publication metadata. A string and a one-id array of that same id are the same request. Discovery gives the eligible account and videos. Duplicate ids collapse. Custom dates are UTC, live, increasing, at most 365 days. First 24 hours, 7 days and 28 days need exactly one video and a sufficiently old publish date. Since published starts at the earliest publish date. Expiry defaults to never.
- The receipt means committed and queued, not generated. Keep its key and payload for safe retries within replay retention, 24 hours by default. A different payload with that key conflicts. The receipt includes `report_id`, pending status, decimal-string `revision`, disclosure and polling instructions, never a share URL or analytics. Poll `agency_get_report` sparingly for current state and selected metrics. Completed, failed, expired and channel_disconnected are terminal for this attempt; an operator may recover delivery failure on the same report later. Request a share URL separately only when asked.
- Report revisions are exact decimal strings. Pass the latest one as `expected_revision` when editing. Editing doesn't refresh or reopen a report: use `agency_refresh_report` or `agency_reopen_report`. The connector can't delete a report.
- `agency_update_report` needs `reports_update`, explicit `agency:read`, `agency:reports:read` and `agency:reports:write` consent, and live Analytics update permission. Confirm the change with the user before calling it. Send only the fields to change: `title`, `brand`, `brand_domain`, `subtitle`, `public_campaign_note`, and supported metrics (`metrics` and `dimensions` on YouTube; `reels_metrics`, `story_metrics` and `dimensions` on Instagram). Account, media, dates and expiration are rejected. Only completed and failed reports can be edited. Pass the revision from the latest `agency_get_report` as `expected_revision`. A stale revision conflicts and writes nothing: read the report again and confirm before a new request. Keep the idempotency key and payload for retries within replay retention, 24 hours by default. A different payload with that key conflicts. A live metric change queues recomputation even during the manual refresh cooldown. Metadata-only and snapshot edits do not. `recomputation` is `queued` or `not_requested` and is separate from the report status. Expired and disconnected reports stay redacted. Poll `agency_get_report` for the new numbers. Do not invent a metric value.
- `agency_refresh_report` and `agency_reopen_report` need `reports_update`, explicit `agency:read`, `agency:reports:read` and `agency:reports:write` consent, and live Analytics update permission. Pass the revision from the latest `agency_get_report` as `expected_revision` and confirm with the user first. Both need an `idempotency_key`. Reuse the same key and payload for retries within replay retention, 24 hours by default. A different payload with that key conflicts. Refresh works only for a live, completed or failed, unexpired report on a connected channel, six hours after its last refresh and below five consecutive failures. Reopen works only for a time-expired report and needs an `expiration_option` the user chose; a live report queues one refresh, a snapshot keeps its stored numbers. A refusal is a `conflict` with `eligibility.reason` (and `next_available_at` for a cooldown): tell the user why and when, do not retry in a loop. The receipt names the new revision and `refresh` (`queued` or `not_requested`), never analytics; poll `agency_get_report` sparingly. Reopening never restores a disconnected channel's data.

- Instagram creation uses `platform: "instagram"`, `reel_ids` / `story_media_ids` and `reels_metrics` / `story_metrics`. Include at least one media item and a metric for each included type; omit metrics for absent types. Only `period: "lifetime"` is accepted; live reporting starts at the earliest publication instant. No date overrides or dimensions. Reel ownership requires fresh account identity and account-feed verification within a configurable 100-page budget. A `service_unavailable` means no creation when proof is incomplete; retry with the same key. Stories must already be collected.
- Read configuration first. Only advertised metric/content combinations and units are supported; Instagram periods are lifetime only.
- List eligible accounts, then media using their `social_account_id`. Require one `type`: `youtube_video`, `instagram_reel` or `instagram_story`.
- One Instagram report can mix reels and collected stories from that account. Browse the two types in separate requests; story `id` is the provider selection ID, `tracking_id` identifies its stored collection.
- YouTube supports `search` and `days_ago` or explicit offset-bearing `published_after`/`published_before`. Reels support explicit bounds; stories support neither search nor publish filters.
- YouTube media pages have a small shared budget. Narrow with `search` or publish bounds before paging far; on `rate_limited`, tell the user and wait for the retry time.
- Follow `next_cursor` even on empty pages, with identical account, type, filters and limit. Default 25; media max 50, accounts max 100. Never pass raw provider cursors, channel IDs, tokens or URLs.
- Legacy accounts without recorded scopes require creator reconnect. Foreign, missing and ineligible accounts all return not found. Provider failures return service unavailable.
- Report reads need `reports_read`. If it's not allowed with `missing_scope`, the user can reconnect and allow Campaign Reports.
- `agency_list_reports` filters by `search`, `platform`, `status` (pending, processing, completed, failed, expired, channel_disconnected) and `expired` (include, exclude or only; default include).
- `analytics_availability: "not_loaded"` marks a completed report whose numbers `agency_get_report` can load. Every other list item shows `"unavailable"` with an `unavailable_reason`.
- `generation_status` pending or processing means the report is still generating. Tell the user and check again later; don't poll in a loop.
- `analytics: null`, `analytics_availability: "unavailable"` or a metric with `availability: "unavailable"` means the value is unavailable. Report its `unavailable_reason` (for example expired, channel_disconnected, not_collected or unsupported_metric). Never estimate a value or show it as zero.
- `agency_get_report_share_link` returns the existing `/report/{id}` URL. Do not build that URL yourself. `share_ready: false` means the public page will not show the report body. Say the `unavailable_reason` and do not present the link as a live report. Pending, processing, or failed does not make a link unready when `share_ready` is true: a failed refresh can leave the previous report on the page. A ready page shows the campaign note to anyone with the link. The call does not publish, revoke, or email.

## Paging

- Lists return `next_cursor`. Null means the last page.
- To continue, call the same tool with identical arguments plus `cursor`. Changing filters, `include` or `limit` invalidates the cursor.
- Cursors expire after about 15 minutes. If a cursor fails with `validation_failed`, restart from the first page.
- Fetch more pages only when the question needs them, and tell the user when more results exist.

## Errors

A tool error carries a `code` and a request id.

- `insufficient_scope`: a permission wasn't granted. The user reconnects and allows it.
- `forbidden`: the user's role doesn't allow it. Reconnecting won't help; an agency admin must change the role.
- `not_found`: the ID isn't in this agency. Missing IDs and other agencies' IDs look the same. Check you passed the right kind of ID.
- `validation_failed`: fix the arguments. Don't retry them unchanged.
- `conflict`: the change can't apply to the current state. Follow its `detail`; don't retry unchanged.
- `rate_limited`: wait the stated seconds, then retry once.
- `service_unavailable`: Availnet or a provider it depends on is unavailable. Retry once shortly. Don't loop.
- `unauthenticated`: the connection lapsed. The user reconnects.
- `internal_error`: retry once. If it persists, give the user the request id for Availnet support.

## Presenting results

- Refer to creators by name and handle. Show IDs only when asked.
- Link creators with `social_accounts[].url`.
- Name the agency, and give `as_of` for inventory.
- Never invent metrics, availability, contacts, rates or prices. The connector never returns contact details, rates, prices or CPM, except a slot's `price_override` and `cpm_override`: report those decimal strings, and say null when they are null. For anything else, say it isn't returned and point the user to Availnet.
- When data is empty, unsupported, null or unavailable, say exactly that.
