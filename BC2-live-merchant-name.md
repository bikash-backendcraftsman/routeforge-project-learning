# BC2 — Soft-uniqueness of live merchant name

**Service:** `catalog-service`  
**API:** `POST /merchants` (create dark store)  
**Status:** Implemented (Java `exists` + V2 unique index + `saveAndFlush`)

---

## What this challenge is

In **one city**, only **one live shop** may use a given **name**.

Live = `OPEN` or `PAUSED`.  
`CLOSED` does **not** keep the name (name can be reused).

Key = **name + cityId** (same name in another city is OK).

Duplicate live name → **409** `RF-CAT-BIZ-002`.

---

## Features implemented

| Feature | Where | Behaviour |
|---|---|---|
| App check before save | `MerchantService.create` | `existsByNameIgnoreCaseAndCityIdAndStatusIn(name, cityId, OPEN+PAUSED)` → 409 |
| Same trimmed values | `create` | `name.trim()`, `cityId` uppercased in **both** exists and `MerchantEntity` |
| Partial unique index | `V2__uk_live_merchant_name.sql` | `UNIQUE (lower(name), city_id) WHERE status IN ('OPEN','PAUSED')` |
| Race mapped to same 409 | `saveAndFlush` + `DataIntegrityViolationException` | Fast duplicate still `BIZ_002` |

Spring Data: `In` in the method name = SQL `IN (...)` → pass `List.of(OPEN, PAUSED)` (`java.util.List`, not `java.awt.List`).

---

## Concepts to remember

**Soft unique** = unique among a **subset of rows**, not every row. A hard `UNIQUE(name, city_id)` would block CLOSED shops from releasing the name.

**Thought process**
1. Unique among whom? → live shops only.  
2. Reject when? → create while a live shop has this name+city.  
3. Allow when? → no live shop, or only CLOSED.  
4. Check before save.

**Chair story (concurrency)**  
`exists` = look at the chair. `save` = sit.  
Two admins can both **look** while it is empty, then both **sit**.  
Java cannot close the gap between look and sit. That is **check-then-act**.

| Layer | Stops |
|---|---|
| Java `exists` | Slow duplicate (B after A committed) |
| V2 unique index | Fast duplicate (A and B in the same second) |

**V1 vs V2**
- V1 `idx_merchants_name_city` = **not unique**, all rows, lookup only. Does **not** enforce BC2.  
- V2 = **UNIQUE** + `WHERE` OPEN/PAUSED. This **is** BC2. Not a duplicate of V1.

**`save` vs `saveAndFlush`**
- `save` = INSERT often delayed until commit; exception can appear late.  
- `saveAndFlush` = INSERT **now** → V2 runs **on that line** → catch and return 409.

The “lock” is a **DB constraint**, not `synchronized`. On merchant insert, V2 (and PK `id`) run — not the SKU unique index.

---

## Interview one-liner

*“exists then save is check-then-act. Two creates can both see free. A partial unique index rejects the second insert. I map unique-violation to the same 409.”*

---

## Related (not this API yet)

**BC5** reopen will reuse this rule with `excludeMerchantId`.  
**BC3** hours / **BC4** timezone belong to hours and `GET /browse`.
