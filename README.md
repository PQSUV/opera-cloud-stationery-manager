# Opera Cloud Stationery Manager

A production-grade automation system for generating, 
managing, and deploying BI Publisher RTF stationery 
templates across a multi-brand, multi-property hotel 
portfolio in Oracle Opera Cloud.

## Scale

- 8 hotel brands (AU, UK, DE, AT, CH, DK)
- 51 properties (33 active in Opera Cloud)
- 169 stationery subtypes
- 963 RTF files per full regeneration
- Multi-language output (EN, DE, FR)

## Architecture

Two systems working together:

**1. Python RTF Builder**
Generates property-specific RTF files from brand 
templates, injecting per-property data (address, 
contact, policy text, checkout times) with full 
multi-language support. Uses a custom visual-layer 
patcher to handle RTF revision-tracking control words 
that break standard string replacement.

**2. Node.js SWARM (Playwright)**
Multi-threaded browser automation that deploys 
generated RTFs directly into Oracle Opera Cloud's 
Manage Reports interface. Three bots:
- Audit bot — scans Opera Cloud across all properties 
  and report groups, writes a master Excel tracking file
- Replace bot — uploads replacement RTFs to existing 
  Opera report entries
- New bot — creates report entries that don't yet exist

## SWARM Architecture

Worker_threads-based multi-threading — each bot runs 
its own event loop and Chrome instance on a dedicated 
CDP port. Eliminates event loop contention from 
Promise.all approaches.

- 3 parallel bots with property-affinity work slices
- Staggered 3s launch to handle Oracle SSO rate limits
- Auto-save Excel every 30s with Ctrl+C graceful exit
- Excel writes on main thread only (worker_threads 
  are not ExcelJS-safe)

Performance: 44 properties in ~10-12 min (vs 25-30 
min single-threaded)

## Tech Stack

- Python — RTF generation, CLI, audit scripts
- Node.js + Playwright — Opera Cloud browser automation
- worker_threads — true parallelism (separate event loops)
- ExcelJS — audit master and reporting
- Oracle Opera Cloud — target deployment platform

## Key Engineering Decisions

**Custom RTF patcher:** Word embeds revision-tracking 
control words mid-string, breaking naive str.replace. 
Built _vis_find / _vis_replace to operate on the 
visual text layer while preserving raw RTF structure.

**Multi-threaded SWARM:** Single-threaded async caused 
bot starvation — every await blocked the shared event 
loop. Worker_threads give each bot true isolation.

**Audit-first workflow:** Before any deployment, the 
audit bot builds a complete picture of what Opera Cloud 
currently has. Replace and New bots operate from this 
master — no blind uploads.

## Outcome

Reduced a multi-day manual stationery management 
process across a 51-property portfolio to a fully 
automated pipeline. RTF generation, compliance 
auditing, and Opera Cloud deployment are all 
unattended operations run from a single CLI.
