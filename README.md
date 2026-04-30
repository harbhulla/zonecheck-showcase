# ZoneCheck

A zoning analysis and due-diligence tool for British Columbia. Search
across 100,000+ parcels and generate citation-backed reports from the
underlying zoning bylaws.

> The source code is private. This repo documents the architecture and
> engineering choices.

---

## What it does

A user enters an address or selects a parcel on the map. ZoneCheck identifies
the parcel's zoning, intersects it with bylaw documents, and generates a
structured report — every claim in the report carries a citation back to a
specific bylaw page, so the output is auditable rather than hallucinated.

## Screenshots

<!-- TODO: add 1–2 map screenshots (parcel + zoning overlays). -->

## Architecture

<!-- TODO: Excalidraw diagram: MapLibre + deck.gl frontend → FastAPI → PostGIS
     (parcels, zoning) + Redis queue + R2 (source documents) + report worker -->

## Tech stack

| Layer | Tools |
|-------|-------|
| Frontend | TypeScript, MapLibre GL, deck.gl, Turf |
| Backend | Python, FastAPI |
| Database | PostgreSQL + PostGIS |
| Queue / cache | Redis |
| Object storage | Cloudflare R2 (bylaw PDFs and source docs) |
| Background work | Worker-driven report lifecycle (pending / running / succeeded / failed) |

## Engineering highlights

### Citation-backed reports

The report pipeline rejects any claim that isn't backed by a citation
(document + page provenance). Extraction failures and partial data fail
loudly into the report state machine instead of silently producing a
half-true report. This was deliberate — for due-diligence work, "we don't
know" is a valid and useful output; "we hallucinated this" is not.

### Geospatial query path

PostGIS handles parcel/zoning intersection at query time. A typical
"what zone is this parcel in" lookup is a single spatial join against
indexed geometries; the same index supports radius queries from the
search box.

### Frontend bundle: 95% Turf reduction

Naive Turf imports pulled in the entire library (~hundreds of kB). I
switched to per-function imports (`@turf/area`, `@turf/intersect`,
etc.) and dropped the Turf-attributable bundle by ~95%.

### Map data: 80% GeoJSON shrink

Raw parcel GeoJSON for visible viewports was tens of MB. I reduced it
~80% by simplifying geometries to viewport-appropriate precision,
quantizing coordinates, and stripping unused properties before serving.
The map stayed visually identical at zoom levels users actually use.

### Coverage

- 100,000+ parcels searchable in the live tool
- 435,000+ parcels in the underlying database

## What I'd do differently

<!-- TODO (Harsimran fills in): 2–3 honest items.
     Examples:
     - Vector tiles (PMTiles) instead of GeoJSON from day one
     - Treat report generation as event-sourced from the start
     - Schema migrations behind feature flags for zoning rule changes -->
