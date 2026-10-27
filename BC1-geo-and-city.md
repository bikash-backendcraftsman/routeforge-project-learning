# BC1 — Geo validity and city containment

**Service:** `catalog-service`  
**API:** `POST /merchants` (create dark store)  
**Status:** Done for v1

---

## What this challenge is

A shop must have a **real map position** and belong to a **city RouteForge actually serves**.

Invalid lat/lng is **bad input** (validation).  
Unknown city is a **business rule** (we do not deliver there).

---

## Features implemented

| Feature | Where | Behaviour |
|---|---|---|
| Name required | `MerchantService.create` | `null` / blank → `RF-CAT-VAL-001` / **400** |
| Lat/lng range + finite | `MerchantService.create` | lat ∈ [-90, 90], lng ∈ [-180, 180], not NaN/Inf → else `VAL_001` / **400** |
| Supported city whitelist | `SupportedCities` + `create` | city not in list → `RF-CAT-BIZ-001` / **422** |
| Hours present | `create` | null/empty map → `VAL_001` / **400** |
| Persist shop | `MerchantRepository.saveAndFlush` | status `OPEN`, `MerchantResponse` |

**v1 design choice:** whitelist of city ids (e.g. BLR, HYD, DEL) — **not** a Bangalore polygon / `ST_Contains`.

---

## Concepts to remember

**Reject-when:** write `if` for the **failure**, then throw. Do not write “when it looks OK” with an empty `{ }`.

**One rule → one `if` → one error.** Name is not city. Geo is not city.

| Rule | Reject when | Code / HTTP |
|---|---|---|
| Geo | missing, not finite, out of range | `VAL_001` / 400 |
| City | not in supported list | `BIZ_001` / 422 |
| Name | null or blank | `VAL_001` / 400 |

**Layers:** HTTP `CreateMerchantRequest` → `CreateMerchantDto` → service → `MerchantEntity` (DB only) → `MerchantResponse`.

**Null vs range:** `latitude == null` means missing. `latitude < -90` means present but invalid. `!= null` in the throw-`if` inverts the rule (throws on valid requests).

**Two designs (not built):** (a) polygon + `ST_Contains` — more honest “inside Bangalore”; (b) city center + max radius. v1 = (b)-like **whitelist**, simpler, can allow a point slightly outside the real city.

---

## Interview one-liner

*“Malformed coordinates are 400 validation. Unsupported city is 422 business. v1 is a city whitelist, not a GIS polygon.”*
