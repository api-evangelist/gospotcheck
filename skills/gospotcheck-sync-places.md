---
name: Sync places into GoSpotCheck
description: Keep the GoSpotCheck place list (locations/accounts) in sync with an external system of record, and organize places into place groups.
api: openapi/gospotcheck-external-openapi.yml
operations: [listPlaces, createPlaces, getPlace, updatePlace, listPlaceGroups, addPlacesToPlaceGroup]
---

# Sync places into GoSpotCheck

Use this to push locations/accounts from your system of record into GoSpotCheck and organize them into place groups.

## Auth
- Send `Authorization: Bearer <token>` on every request (token from your GoSpotCheck CSM / support@gospotcheck.com).
- Base URL: `https://api.gospotcheck.com`.

## Steps
1. **Check what exists** — `listPlaces` (paginate with `page`/`per_page`, max 200). Filter to find a known place, e.g. `?custom_place_id.eq=<your-id>`.
2. **Create places** — `createPlaces` (POST `/external/v1/places`). Send a single object or an array to batch-create. Set `custom_place_id` to your external key so you can reconcile later; include `name`, `address`, `city`, `state`, `postal_code`, `country` (ISO 3166-1 alpha-2).
3. **Update a place** — `getPlace` then `updatePlace` (PUT `/external/v1/places/{id}`) for address/name changes.
4. **Group places** — `listPlaceGroups`, then `addPlacesToPlaceGroup` (PUT `/external/v1/place_groups/{id}/add_places?place_ids=1,2,3`).

## Rules
- Rate limit: 10 req/s (serial, no concurrency) and 100,000/day; over-limit returns `403 Forbidden (Rate Limit Exceeded)` — back off.
- No idempotency key exists; reconcile on `custom_place_id` before creating to avoid duplicates.
- Requests time out after 30s. For large reads add `_async=true` to get a CSV export job instead.
- Errors: non-2xx responses populate the `errors` hash; `422` means validation failed.
