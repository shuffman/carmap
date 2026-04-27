# CarMap

A client-side web app that parses a CARFAX vehicle history report PDF and plots the car's full history on an interactive map.

Upload a CARFAX PDF and CarMap will extract every recorded event — ownership changes, service visits, accidents, sales, and inspections — geocode each location, and display them as color-coded markers on a map connected by the car's travel path.

---

## Features

- **PDF parsing** — reads CARFAX report PDFs directly in the browser using [pdf.js](https://mozilla.github.io/pdf.js/); no server, no upload
- **Event classification** — automatically identifies ownership changes, service records, accidents, sales/listings, and emissions/safety inspections
- **Interactive map** — markers color-coded by event type; polylines trace the car's geographic journey, colored by owner
- **Ownership-aware routing** — lines between locations are colored per owner, with dashed segments marking ownership transitions
- **Filterable sidebar** — scrollable event timeline with per-type filters; clicking a sidebar row pans to that marker and vice versa
- **Geocoding** — locations are resolved to coordinates using a pre-seeded cache of common US cities with [Nominatim](https://nominatim.openstreetmap.org/) as fallback
- **No API key required** — uses OpenStreetMap tiles and Nominatim; fully free

---

## Usage

1. Open `index.html` in any modern browser (Chrome, Safari, Firefox, Edge)
2. Drag and drop a CARFAX report PDF onto the upload area, or click to browse
3. CarMap extracts and parses the report, geocodes the locations, and renders the map

No installation, build step, or server required — it's a single HTML file.

---

## Map Legend

### Event types

| Color | Type | Triggered by |
|-------|------|-------------|
| 🔵 Blue | Ownership change | "New owner reported", "Title issued or updated", "Vehicle purchase reported" |
| 🟢 Green | Vehicle serviced | "Vehicle serviced", "Maintenance inspection", "Oil and filter changed", "Pre-delivery inspection" |
| 🔴 Red | Accident | "Accident reported", "Damage Report" |
| 🟡 Amber | Sale / Listing | "Vehicle offered for sale", "Vehicle sold", "Sold as a Certified Pre-Owned" |
| 🟣 Purple | Inspection | "Passed emissions inspection", "Passed safety inspection" |
| ⚫ Gray | Other | Registration renewals, odometer readings, etc. |

### Map lines

Polylines connect events in chronological order. Each segment is colored by the **current owner** at the time of the event. A **dashed segment** indicates an ownership transition between two locations.

---

## How It Works

### PDF text extraction

CarMap uses pdf.js to read the PDF page by page. Text items are grouped by their Y-coordinate (quantized to 2px buckets) to reconstruct the visual line layout of the CARFAX table, then sorted left-to-right within each line.

### Parsing

The parser scans the reconstructed lines for:

- **Owner headers** — lines matching `Owner N` (standalone) mark the start of each owner's section and enable event recording once `Owner 1` is seen (preventing false positives from summary tables earlier in the report)
- **Date lines** — lines beginning with `MM/DD/YYYY` start a new event record; optional mileage follows immediately after the date
- **Locations** — lines matching `City, ST` (two-letter uppercase state code) are extracted as the event's location; a fallback embedded match handles lines where the location appears alongside a source name (e.g. `California Motor Vehicle Dept. Palm Desert, CA`)
- **Comments** — all remaining lines within an event block are collected and used for event classification and popup display
- **Termination** — parsing stops when the Glossary, disclaimer, or footer is detected, preventing boilerplate text from bleeding into the last event's record

Events without an explicit location inherit the most recent known location from a preceding event (used for entries like accident damage reports, which have no associated address).

### Geocoding

Unique city/state pairs are resolved to lat/lng coordinates using a pre-seeded cache of ~50 common US cities. Any location not in the cache is looked up via the Nominatim API (rate-limited to one request per second per their usage policy). Results are cached in memory for the session.

---

## Tech Stack

| Library | Version | Purpose |
|---------|---------|---------|
| [pdf.js](https://mozilla.github.io/pdf.js/) | 3.11.174 | PDF text extraction |
| [Leaflet](https://leafletjs.com/) | 1.9.4 | Interactive map |
| [OpenStreetMap](https://www.openstreetmap.org/) | — | Map tiles |
| [Nominatim](https://nominatim.openstreetmap.org/) | — | Geocoding fallback |

No build tools, no framework, no dependencies to install.

---

## Limitations

- **CARFAX format only** — the parser is tuned to CARFAX's specific PDF layout. Reports from other providers will not parse correctly.
- **City-level precision** — CARFAX records locations at the city/state level, so map pins represent the city center, not a specific address.
- **Unreported events** — as CARFAX itself notes, not all accidents or service events are reported; CarMap can only show what's in the report.
- **Nominatim rate limit** — the free Nominatim API allows one request per second. Reports with many unique, uncached cities may take a few extra seconds to geocode.

---

## Privacy

Everything runs locally in your browser. No data from the PDF is transmitted to any external server except for geocoding requests (city + state name only, no VIN or personal information) sent to Nominatim when a location is not already in the local cache.
