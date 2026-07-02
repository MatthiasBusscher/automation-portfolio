# Multi-Administration Cost Center Automation (Pipedrive → Exact Online)

> **Context** Multi-administration finance setup · operational and financial records need consistent object-level tracking across administrations
> **Stack** Pipedrive · Make.com · Exact Online API (raw HTTP, OAuth2 from the [token management system](03-oauth2-token-management.md))
> **Category** Finance automation & ERP integration

## The problem

Reliable object-level reporting requires related operational and financial records to share the same cost center (object code) in Exact Online across multiple administrations. Every new object meant finance manually creating that cost center more than once, with names that had to match exactly. Skipped or mistyped cost centers meant downstream reporting and allocation became unreliable.

## Architecture

```mermaid
flowchart LR
    PREV["Order-to-cash flow<br/>writes debtor number"] --> PD["Pipedrive detects<br/>debtor number set"]
    PD -- webhook --> MAKE["Make.com scenario"]
    TOKENS[("OAuth2 tokens<br/>(Data Store)")] -.-> MAKE
    MAKE --> C1{"Cost center exists<br/>in admin A?"}
    MAKE --> C2{"Cost center exists<br/>in admin B?"}
    C1 -- no --> CREATE1["Create via raw HTTP<br/>(administration A)"]
    C2 -- no --> CREATE2["Create via raw HTTP<br/>(administration B)"]
    CREATE1 & CREATE2 --> WB["Write object codes<br/>back to CRM"]
```

A domino architecture: completion of the debtor-creation flow (which writes the WeFact debtor number to the CRM) is itself the trigger for this one. The scenario authenticates with stored OAuth2 tokens, checks *per administration* whether the cost center exists, creates whatever is missing in parallel, and reports the object codes back to the CRM.

## Key decisions & trade-offs

- **Chained processes ("domino") vs. one mega-flow.** Each step (deal → debtor → cost centers) is a separate scenario triggered by the previous one's written-back result. This keeps every flow small, independently testable, and re-runnable — at the cost of the chain's state living in CRM fields rather than a single orchestrator. For this team size, debuggability won.
- **CRM as the master for naming.** One object name entered in Pipedrive propagates to identical cost-center structures across multiple Exact administrations. The alternative (finance naming cost centers in Exact) was the status quo that produced mismatches.
- **Check-then-create per administration.** Each administration is independent — one can already have the cost center while another does not. Treating "Exact" as one target would have made re-runs unsafe; per-administration idempotency makes the flow safe to fire twice.
- **Raw HTTP over connector modules.** Exact's cost-center endpoints and multi-administration switching needed more control than the standard connector exposed; raw requests with manually managed tokens were the reliable path.

## The hardest part

Multi-administration authentication and routing. Every API call must target the right Exact *division* (administration) with a valid token — against tokens that rotate weekly via a separate system, and division IDs that differ per target administration. Composing this from stored tokens + division IDs from configuration, inside Make's HTTP modules, with per-administration existence checks, is where most of the engineering time went.

## Results

- Reliable object-level reporting: related records land on the same cost center automatically, in each required administration.
- The full chain — new deal → debtor → cost centers — runs without any manual finance work.
- One name in the CRM produces an identical, correct data structure across multiple Exact Online administrations, every time.
- The correct object code is available in each required administration from day one, so downstream financial allocation works without manual correction.

## Limitations & what I'd do differently

- The domino chain's state lives in CRM fields, so a human clearing or editing those fields can re-trigger or break the chain — guard conditions mitigate but don't eliminate this. With more flows, I'd introduce an explicit orchestration/state layer.
- Renaming an object after creation doesn't propagate to Exact — confirmed create-once; the flow is chained off initial debtor creation and has no update trigger, so renames in the CRM don't reach Exact.
- Failure visibility relied on Make's error handling, which wrote a diagnostic note to the Pipedrive deal identifying exactly which step broke — making the failure visible in the CRM and the deal manually retryable. For a finance-critical chain I'd now add proactive alerting rather than waiting for someone to notice the note.
