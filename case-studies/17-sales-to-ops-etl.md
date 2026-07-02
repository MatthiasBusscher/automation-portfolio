# Sales-to-Operations Data Pipeline (ETL)

> **Context** Handover from sales (Pipedrive) to operational planning (Google Sheets backend) on every won deal
> **Stack** Pipedrive · Make.com · Google Shared Drive · Sheets (`_ImportData` staging tab)
> **Category** Data pipelines (ETL) & CRM automation

## The problem

A won deal triggers the real work: customer records, product line-items, planning attributes, and supporting documents must travel from the CRM into the operational planning systems. Done manually, this handover corrupted data exactly where it hurt — missing integration IDs, mistyped quantities, and related documents lost in inboxes. Three things made naive automation insufficient: deals could be marked "won" before upstream administrative IDs existed; deals carry *nested* multi-product data that a flat spreadsheet row can't hold; and some handovers required a supporting file to move along with the data.

## Architecture

```mermaid
flowchart LR
    PD["Pipedrive deal won"] --> GATE{"Gatekeeper trigger:<br/>won AND CostCenter<br/>AND DebtorCode present?"}
    GATE -- no --> WAIT["No fire — waits for<br/>upstream chain to complete"]
    GATE -- yes --> ROUTE{"Contract type?"}
    ROUTE -- "Subscription /<br/>One-Time" --> EXTRACT["API: fetch all<br/>product line-items"]
    ROUTE -- "Documented handover" --> FILE["Download supporting file<br/>→ archive in Shared Drive"]
    FILE --> EXTRACT
    EXTRACT --> TRANSFORM["Iterate + aggregate:<br/>clean JSON object<br/>(SKU, qty, line metadata)"]
    TRANSFORM --> LOAD[("Load into _ImportData<br/>tab of ops backend")]
```

A strict gatekeeper webhook fires only when a deal is won *and* carries both required integration IDs — i.e., only after the upstream [debtor](06-order-to-cash-automation.md) and [cost-center](07-multi-entity-cost-centers.md) flows have done their work. The flow then routes by handover type, extracts and transforms nested product data into a clean JSON object, archives any required supporting file to the operational Drive, and loads everything into the planning backend's staging tab.

## Key decisions & trade-offs

- **Gatekeeping at the source vs. validating at the destination.** Rejecting incomplete deals at the trigger means operations *never* sees a partial dossier — there's nothing to clean up, chase, or accidentally plan from. The trade-off: a deal missing its IDs fires nothing and waits silently, which demands trust in the upstream chain. When a Make step failed mid-run, a diagnostic note was written to the Pipedrive deal — but deals that never triggered at all (still waiting on IDs) were invisible without actively checking the CRM.
- **JSON-in-a-cell as the transport format for line-items.** A deal's products are a variable-length nested structure; flattening to columns either truncates or explodes the schema. Aggregating line-items into one clean JSON string (SKU, quantities, line metadata) keeps the staging interface stable while the GAS backend parses it into whatever structure planning needs. Unorthodox, deliberate, and it worked — the schema never had to change for product-count reasons.
- **Files travel with the data.** When supporting documents are required, the file is downloaded from the CRM and archived in the correct Shared Drive in the same run — the handover is atomic from operations' perspective: data *and* paperwork arrive together or not at all.
- **A staging tab (`_ImportData`) as the contract between systems.** Make's responsibility ends at a well-defined drop-off point; the [ERP engine](18-financial-erp-refactor.md) takes over from there. Clean separation made both sides independently debuggable.

## The hardest part

The transform step. Pipedrive returns deal products as nested API resources that must be fetched separately, iterated, enriched with stable line metadata, and aggregated into one syntactically valid JSON string using Make's iterator/aggregator semantics — where escaping, separators, and empty-product edge cases all conspire against you. Producing JSON that *always* parses on the GAS side, for every deal shape sales could produce, took most of the build time.

## Results

- Operations never receives an incomplete customer dossier — completeness is enforced before transmission, not checked after.
- Multi-line product structures arrive as clean, parseable data; mistyped quantities and lost integration IDs are gone from the handover.
- Supporting documents are archived in the right operational location automatically, in the same transaction as the data.
- Handover lead time from signature to plannable: effectively zero, with no human touchpoints.

## Limitations & what I'd do differently

- Silent gatekeeping is correct but opaque: a deal stuck missing one ID looks identical to a deal nobody won. No "blocked deals" digest was built — this remains a real operational gap where a digest would add meaningful visibility.
- JSON-in-a-cell trades queryability for schema stability — fine as a transport format, but it makes the staging tab itself unreadable to humans; a small rendered preview column would have helped support.
- Changes to a deal *after* handover (upsells, corrections) were confirmed out of scope and handled manually — the natural next iteration is an update event alongside the create event.
