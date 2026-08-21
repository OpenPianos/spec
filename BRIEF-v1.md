# OpenPianos — Product & Technical Brief (v1)

> Historical document, preserved as-is. Written by **Daniel Seixas** (PianoMeetups), August 2026 —
> the founding brief credited in `README.md`. Some specifics have since evolved (stack: the
> MariaDB/Fastify plan became a Cloudflare Worker + Neon Postgres; workflow: pre-moderation became
> the wiki model of ADR-0005) — see `DECISIONS.md`. The dataset/API/contribution pillars, stable
> IDs, statuses, export channels, and the GitHub dataset mirror structure remain the blueprint.

## Executive Summary

**OpenPianos** is an open-data initiative whose mission is to become the **canonical source of truth for publicly accessible pianos worldwide**.

Rather than creating yet another piano map, OpenPianos provides:

- A shared dataset
- A public API
- Open exports
- A contribution workflow

allowing multiple applications to consume and contribute to the same data.

Potential consumers include:

- Plinkato
- PianoMeetups
- pianos.pub
- Mobile apps
- Travel guides
- Open-data projects
- Future community initiatives

The long-term objective is for the ecosystem to evolve from:

```
Multiple sites
Multiple databases
Duplicate effort
```

to:

```
One dataset
Many applications
```

---

# Business Goals

## Primary Goal

Create and maintain the world's most complete open database of public pianos.

---

## Secondary Goals

### Data Portability

The data should never be locked into a specific application.

Consumers must be able to access:

- API endpoints
- Public exports
- Historical changes

without requiring permission from OpenPianos.

---

### Community Ownership

The project should be perceived as community infrastructure rather than a commercial platform.

The dataset should remain usable even if:

- OpenPianos changes ownership
- A consuming application disappears
- New applications enter the ecosystem

---

### Ecosystem Growth

OpenPianos should encourage third-party innovation.

Examples:

- Piano discovery apps
- Piano meetup platforms
- Travel integrations
- Music community projects

---

# Product Strategy

OpenPianos is not primarily a website.

OpenPianos is:

```
Open Dataset
      +
Public API
      +
Community Contribution Layer
```

The website is merely one possible consumer.

---

# Open Data Strategy

## Public API

Primary integration mechanism.

```
Consumers
     ↓
OpenPianos API
```

---

## Public Data Exports

To guarantee long-term accessibility, the complete dataset will be exported regularly.

Supported formats:

- JSON
- GeoJSON
- CSV

These exports are publicly downloadable.

Example:

```
https://openpianos.org/export/pianos.json
https://openpianos.org/export/pianos.geojson
https://openpianos.org/export/pianos.csv
```

---

## GitHub Mirror

A public GitHub repository acts as a permanent, versioned archive of the dataset.

Example:

```
github.com/openpianos/dataset
```

Exports are automatically published on a schedule.

Example:

```
daily
weekly
after approved changes
```

This provides:

- Transparency
- Version history
- Backup
- Independent reuse
- Public trust

The GitHub repository is **not** the primary database.

It is an open mirror of the canonical dataset.

Architecture:

```
MariaDB
    ↓
Export Pipeline
    ↓
GitHub Repository
```

---

# Governance Principles

## Stable Identifiers

Every piano receives a permanent ID.

Example:

```
pno_4f1e9ab7
```

IDs never change.

This allows multiple applications to reference the same piano consistently.

---

## Data Preservation

Pianos are never physically deleted.

Instead:

```
active
temporary
needs_verification
removed
```

Historical records remain available.

---

## Attribution & Transparency

All modifications generate change records.

Consumers can inspect:

- Creation history
- Updates
- Verifications
- Removal reports

---

# Contribution Workflow

## Piano Submission

```
Submit
   ↓
Pending Review
   ↓
Approved
   ↓
Published
```

---

## Verification Workflow

```
User visits piano
        ↓
Verify existence
        ↓
Verification stored
        ↓
Last verified updated
```

---

## Removal Workflow

```
Report missing piano
         ↓
Review
         ↓
Status updated
```

No data is permanently deleted.

---

# Technical Architecture

## Technology Stack

### Backend

```
Node.js
TypeScript
Fastify
Drizzle ORM
```

### Database

```
MariaDB
```

### Source Control

```
GitLab (Hyperthings private gitlab)
```

### Future Integrations

Reserved for:

```
OpenStreetMap
Redis
```

but not required for MVP.

---

# Data Model

## Piano

Core entity.

Fields:

```
id
name
latitude
longitude

countryCode
city

accessType
feeRequired

indoor

status

description

createdAt
updatedAt
lastVerifiedAt

osmId (optional)
```

---

## Verification

Stores community confirmations.

```
id
pianoId

result

notes

verifiedAt
```

---

## Photos

Stores piano imagery.

```
id
pianoId

url

uploadedAt
```

---

## Change Log

Tracks all modifications.

```
id

entityType
entityId

action

createdAt
```

---

# API Strategy

Base URL:

```
/api/v1
```

---

## Public Endpoints

### List Pianos

```
GET /pianos
```

Examples:

```
GET /pianos?country=ES
GET /pianos?city=Bilbao
GET /pianos?status=active
```

---

### Piano Details

```
GET /pianos/{id}
```

---

### Nearby Search

```
GET /pianos/nearby
```

Parameters:

```
lat
lon
radius
```

---

### Change Feed

```
GET /changes
```

Allows consumers to synchronize updates.

---

## Contribution Endpoints

### Create Piano

```
POST /pianos
```

---

### Update Piano

```
PATCH /pianos/{id}
```

---

### Verify Piano

```
POST /pianos/{id}/verify
```

---

### Upload Photo

```
POST /pianos/{id}/photos
```

---

# Data Distribution

OpenPianos distributes data through three channels.

## 1. Real-Time API

Best for applications.

```
Plinkato
PianoMeetups
pianos.pub
```

---

## 2. Downloadable Exports

Best for researchers and developers.

Formats:

```
JSON
GeoJSON
CSV
```

---

## 3. GitHub Dataset Mirror

Best for:

- Transparency
- Community trust
- Historical snapshots
- Forkability

Example structure:

```
dataset/
├── latest/
│   ├── pianos.json
│   ├── pianos.geojson
│   └── pianos.csv
│
├── snapshots/
│   ├── 2026-08-15/
│   ├── 2026-08-16/
│   └── ...
│
└── schema/
```

This enables consumers to:

- Download the latest dataset
- Inspect historical versions
- Compare changes over time

without needing access to the production database.

---

# MVP Deliverables

## Core Platform

- Piano management
- Verification workflow
- Photo support
- Change tracking

---

## Public API

- List
- Detail
- Nearby search
- Verification
- Change feed

---

## Open Data

- JSON export
- GeoJSON export
- CSV export
- GitHub mirror

---

## Community Foundation

- Stable IDs
- Open access
- Transparent history
- Public documentation

---

# Success Criteria

OpenPianos succeeds when:

1. Multiple piano-related applications consume the dataset.
2. Contributors can update data without contacting individual websites.
3. The dataset remains accessible independently of any single frontend.
4. New projects choose OpenPianos as their piano data provider rather than creating a separate database.

At that point, OpenPianos becomes infrastructure for the public-piano ecosystem rather than simply another piano map.
