# Database fields expected by this workflow

This is a **reference**, not a full migration script. Review existing keys, types, defaults, and constraints before applying changes.

## `companies`

- `id` (bigint, primary identifier)
- `company_name` (text)
- `active` (boolean)

## `observations`

- `id` (generated bigint identifier)
- `company_id` (bigint)
- `source_type` (text)
- `source_url` (text)
- `title` (text)
- `content` (text)
- `published_at` (timestamp with time zone)

The `Save New Observations` query expects a unique constraint or unique index on `(company_id, source_url)`.

## `events`

- `id` (generated bigint identifier)
- `company_id` (bigint, required)
- `observation_id` (bigint, optional)
- `event_type` (text, required)
- `title` (text)
- `summary` (text)
- `event_date` (date)
- `confidence` (numeric)
- `status` (text, defaults to `candidate` in the query)
- `reasoning` (text)
- `created_at` (timestamp with time zone)
- `matched_keywords` (jsonb)
- `classification_method` (text)
- `is_significant` (boolean)

The event-saving SQL uses query parameters to store candidates and skip records with an existing `observation_id`. For concurrency-safe duplicate prevention, consider a unique index on non-null `observation_id` after verifying current data.
