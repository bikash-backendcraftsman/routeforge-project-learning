# RouteForge Project Learning

Personal notes while building **RouteForge**, starting with **catalog-service**.

## Catalog — `POST /merchants`

| File | Challenge | Status |
|---|---|---|
| [BC1-geo-and-city.md](./BC1-geo-and-city.md) | Geo validity and supported city | Done (v1 whitelist) |
| [BC2-live-merchant-name.md](./BC2-live-merchant-name.md) | Soft-unique live shop name (OPEN/PAUSED) | Done (`exists` + V2 unique index) |

Still on this API: BC3 hours, BC4 timezone / browse, BC5 reopen.

Implementation lives in `routeforge-catalog-service`.
