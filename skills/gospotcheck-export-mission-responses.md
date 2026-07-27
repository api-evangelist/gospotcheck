---
name: Export mission and task response data
description: Pull GoSpotCheck mission and task response data out for custom reporting, using synchronous paging or asynchronous CSV export for large datasets.
api: openapi/gospotcheck-external-openapi.yml
operations: [listMissions, listMissionResponses, listTaskResponses, getAsyncCsvExportStatus]
---

# Export mission and task response data

Use this to extract survey/task ("mission") results from GoSpotCheck for BI or a data warehouse.

## Auth
- `Authorization: Bearer <token>`; base URL `https://api.gospotcheck.com`.

## Steps
1. **Find missions** — `listMissions`. Filter to the campaign(s) you care about, e.g. `?state.eq=completed` or `?id.in=4,3,2`.
2. **Read mission responses** — `listMissionResponses` (`/external/v1/mission_responses`, read-only). Each response links `mission_id`, `place_id`, `user_id`. Use `include=` for related data.
3. **Read task responses** — `listTaskResponses` (`/external/v1/task_responses`, read-only) for per-question answers; join to mission responses via `mission_response_id`.
4. **Large exports** — append `_async=true` (optionally `_delimiter` and `_headers`) to an index call. The response returns a `job_guid`; poll `getAsyncCsvExportStatus` (`/external/v1/async_jobs/csv_exports/{job_guid}/status`) until `download_url` is populated, then fetch the CSV.

## Rules
- Rate limit 10 req/s serial, 100,000/day; over-limit = `403 (Rate Limit Exceeded)`.
- Sync index calls cap at 200 records/page — use async CSV for full extracts.
- Async CSV files expire one week after creation; download promptly.
- `include`/`methods` are ignored inside async requests.
- Filter timestamps as unix seconds with `.gt`/`.lt` operators (e.g. `created_at.gt=1635283293`).
