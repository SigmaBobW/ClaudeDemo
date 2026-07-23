# IEEE Membership Model

**Data model ID:** 2e1295a0-aa69-4640-adfc-8bf1c5e3505b
**URL:** https://app.sigmacomputing.com/ieee/data-model/IEEE-Membership-Model-1oW8GPkJhbA5P1RVqxEun1
**Folder:** User Folder (03308095-2e57-4967-9127-9b31e83a6634)
**Connection:** dbx_dev (41e2542e-133f-4d7d-aaa5-5b5416a23045)

## Elements
- `memberships` — warehouse-table: dp_it_catalog.dp_it_views.t_memberships_subscriptions_data (raw fact, 61 cols)
- `active-contacts` — custom SQL: contacts filtered to status_description = 'Active' (107 cols)
- `membership-detail` — custom SQL: memberships LEFT JOIN active-contacts ON contact_id = customer_number (keeps all memberships)

## Decisions (user-approved)
- Join key: memberships.contact_id = contact.customer_number
- Join semantics: LEFT (all memberships retained; contact fields null when no Active contact matches)
- "Active only" enforced by pre-filtering contacts in SQL subquery

## Build notes / gotchas discovered
- Created via Code Representation API: POST /v2/dataModels/spec (schemaVersion 1).
- Sigma native `join` source resolves join-column refs ONLY against inline `warehouse-table`
  sources (live-schema introspection); it CANNOT resolve refs to `elementId` references
  (element base columns aren't in the code-rep namespace). Since the Active filter requires a
  pre-filtered contacts source, a native join could not reference it -> used a custom-SQL
  element for the join instead. Column-name overlap between the two tables is NONE, so
  `SELECT m.*, c.*` is collision-free.
- Column refs in native joins use DISPLAY names in brackets: `[Contact Id]`, not `[contact_id]`.
- Code-rep-created elements report 0 columns via API (/columns, describe, query) until the
  model is opened in the UI, which lazily resolves the source schema. Therefore the join-key
  match rate could NOT be runtime-validated via API. TODO: confirm contact match rate in the UI
  (or re-query once columns materialize). LEFT join means memberships are never dropped even if
  customer_number turns out to be the wrong physical key.
