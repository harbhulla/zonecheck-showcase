# ZoneCheck

A zoning analysis and due diligence tool for British Columbia. You can search across 100,000+ parcels and get back citation-backed reports built from the underlying zoning bylaws.

The source code is private. This repo is here to document how I built it.

## What it does

You enter an address or click a parcel on the map. ZoneCheck identifies the parcel's zoning, intersects it with the relevant bylaw documents, and generates a structured report. Every claim in the report carries a citation back to a specific bylaw page, so you can actually verify what you're reading instead of taking the tool's word for it.

## Screenshots

<!-- TODO: 1 or 2 map screenshots showing parcel and zoning overlays -->

## Architecture

<!-- TODO: Excalidraw diagram. MapLibre + deck.gl frontend, FastAPI backend, PostGIS for parcels and zoning, Redis queue, Cloudflare R2 for source documents, report worker -->

## Tech stack

| Layer | Tools |
|-------|-------|
| Frontend | TypeScript, MapLibre GL, deck.gl, Turf |
| Backend | Python, FastAPI |
| Database | PostgreSQL with PostGIS |
| Queue / cache | Redis |
| Object storage | Cloudflare R2 (bylaw PDFs and source docs) |
| Background work | Worker-driven report lifecycle (pending, running, succeeded, failed) |

## Engineering highlights

### Citation-backed reports

The report pipeline rejects any claim that isn't backed by a citation (document plus page number). If extraction fails or data is partial, the report state machine fails loudly instead of papering over it. For due diligence work I'd rather have the tool say "we don't know" than spit out a confident half-truth.

### Geospatial query path

PostGIS does most of the heavy lifting. A parcel-to-zoning lookup is a single spatial join against indexed geometries, and the same index handles radius queries from the search box.

### Cutting the Turf bundle by 95%

The naive Turf imports were pulling in the whole library, hundreds of kB of code I wasn't using. I switched to per-function imports (`@turf/area`, `@turf/intersect`, and so on), and the Turf-attributable bundle dropped by about 95%.

### Shrinking GeoJSON by 80%

Raw parcel GeoJSON for the visible viewport was tens of MB. I cut it by about 80% by simplifying geometries to viewport-appropriate precision, quantizing coordinates, and dropping unused properties before serving. At the zoom levels people actually use, the map looks the same.

### Coverage

- 100,000+ parcels searchable in the live tool
- 435,000+ parcels in the underlying database
