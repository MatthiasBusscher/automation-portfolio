# Product Master Data Sync (Pipedrive & WeFact)

> **Context** B2B services group with a complex operating structure · products = services and physical deliveries
> **Stack** Google Apps Script (webhook trigger) · Make.com · Pipedrive · WeFact
> **Category** Master data management & API integration

## The problem

Every new or changed product had to be entered manually in several connected systems. With no single source of truth, teams worked from outdated catalogs and invoices could carry inconsistent product data. Product changes happened periodically and when supplier catalog updates were communicated. The administration structure added real complexity: some product types needed to exist in more than one finance administration, while others belonged to a single administration. Keeping that straight by hand was unrealistic.

## Architecture

```mermaid
flowchart LR
    DB[("Central product DB<br/>(Sheets log)")] -- "product added/changed" --> GAS["GAS webhook trigger"]
    GAS --> MAKE["Make.com sync flow"]
    MAKE --> SEARCH{"Product exists<br/>in target system?"}
    SEARCH -- yes --> UPD["Update (upsert)"]
    SEARCH -- no --> CREATE["Create"]
    UPD & CREATE --> ROUTE{"Product type?"}
    ROUTE -- "type A" --> BOTH["Finance system:<br/>selected administrations"]
    ROUTE -- "type B" --> SINGLE["WeFact: single<br/>administration"]
    MAKE --> PD["Pipedrive catalog"]
```

A change in the central product database fires a GAS webhook into Make.com. The flow upserts the product into Pipedrive and WeFact, with conditional routing deciding *which* WeFact administrations get the product based on its type.

## Key decisions & trade-offs

- **One designated master.** The product database is the only place where products are created or edited; CRM and invoicing are read-only consumers. This deliberately sacrifices flexibility (sales can't add ad-hoc products in Pipedrive) to guarantee consistency — the precise failure mode we were eliminating.
- **Upsert instead of create-only.** Idempotent sync: replaying an event or re-syncing a product is less likely to produce duplicates. This made the flow safer to re-run after partial failures.
- **Webhook from the database vs. Make polling the sheet.** Push keeps latency low and avoids burning Make operations on empty polls; the GAS side already had an edit-log to hook into.
- **Routing logic in the flow, not in the data.** Product type drives administration routing centrally in Make. The alternative — flagging target administrations per product in the database — was rejected as one more manual field to get wrong.

## The hardest part

Finance-administration routing. Some products need to exist, with consistent identifiers, in more than one finance administration. Getting the create/update logic idempotent across the required targets, where one might already have the product and another might not, required checking and branching per administration rather than treating the finance system as a single target.

## Results

- Invoicing errors from stale product names, outdated product data, or wrong categories were reduced by syncing from the shared catalog.
- Duplicate data entry across connected systems was reduced.
- New products are available to sales and administration immediately after a single entry.
- The finance-administration product structure is created consistently without anyone having to remember the routing rules.

## Evolution: product group data moved out of Make (cost & maintainability)

The original sync resolved each product's group via a **per-record lookup against a Make.com data store** — costing Make operations on every record, and meaning product groups could only be maintained inside the Make platform.

The reference data was relocated into the product database. Product groups now live in a dedicated sheet keyed by a **category code** — the same code carried on the product records and written to the logbook sheet. During the run, the Apps Script matches on the category code across these sheets, resolves the correct product group ID, and includes it directly in the payload sent to the Make webhook. Make no longer performs any data store lookup.

**Result:**
- The per-record data store lookup was removed, lowering recurring Make operations cost.
- A new product group is added by editing a sheet instead of the Make data store — accessible to non-technical staff, and consistent with the config-driven approach used across the rest of the system.

**What I'd add:** validation on the category code (check against an allowed list) so a typo in the sheet can't pass an invalid group through the sync.

## Limitations & what I'd do differently

- The sync is one-directional. If someone edits a product directly in Pipedrive or WeFact, the change silently diverges until the next sync overwrites it there's no conflict detection. Locking down edit rights in the consumer systems was the pragmatic mitigation.
- Deletions or setting products to inactive was currently out of scope.
- Today I would add a periodic reconciliation run (compare master against consumers, report drift) as a safety net under the event-driven sync.
