# Work Schedule Configurator (Interactive UI in Google Sheets)

> **Context** Operational planning · composing structured work schedules from valid object/area/item/action combinations
> **Stack** Google Apps Script · Google Sheets (as application UI) · CacheService · LockService · PropertiesService
> **Category** Internal tools & UI engineering

## The problem

The work schedule was a large matrix: which object type, which area types, which items in those areas, and which actions per item. Users assembled these by copying from large source documents — slow, and structurally error-prone in a specific way: nothing prevented invalid combinations from reaching output documents. A bespoke web application would solve it but was too costly and slow to build; the constraint was to deliver configurator-grade UX inside Google Sheets without the browser grinding to a halt.

## Architecture

```mermaid
flowchart LR
    SRC[("External source DB:<br/>object/room/element/<br/>action matrix")] --> CACHE["CacheService:<br/>async-cached dataset"]
    USER["User checks ObjectType"] --> EDIT["onEdit trigger<br/>(LockService-guarded)"]
    EDIT --> CASCADE["Cascading filters:<br/>clear screen, render only<br/>valid next-level checkboxes"]
    CASCADE --> L2["RoomTypes → Elements<br/>→ Actions"]
    L2 --> PUSH["'Push': export combinations<br/>to output template"]
    PUSH --> UNDO["undoLastPush:<br/>coordinates stored in<br/>PropertiesService"]
```

The sheet behaves like a four-level cascading configurator: each selection clears the levels below and dynamically renders only the checkboxes that are valid according to the source matrix. The heavy source dataset is cached for instant response; a guarded `onEdit` trigger drives the interaction; a one-click push exports the configured matrix (sometimes hundreds of rows) to the output template — with a one-click undo.

## Key decisions & trade-offs

- **Sheets as the app platform.** The honest comparison wasn't "Sheets vs. a nice web app" but "Sheets vs. no tool this quarter." Sheets brought zero hosting, zero login management, and a UI the users already trusted. The trade-off: building UI behavior (cascading filters, dynamic controls) on a platform with no native support for it — which is exactly where the engineering interest lies.
- **Validity by construction, not validation after.** The UI only ever *renders* valid options — invalid combinations aren't rejected, they're unrepresentable. This moves the source matrix into the role of single source of truth and eliminates the review burden on output documents.
- **`CacheService` between the UI and the source database.** Reading thousands of matrix rows on every click would make the interaction unusable. Caching the dataset in script cache turns each cascade step into an in-memory filter — the difference between a "tool" and a "toy."
- **Undo via `PropertiesService`.** `undoLastPush` stores the last export's target coordinates, so a wrong push into an output template is reversible with one click instead of a manual cleanup in a delivery document. Small feature, outsized trust effect: people use a push button much more freely when there's an undo.
- **`LockService` on `onEdit`.** Rapid clicking fires overlapping trigger executions that corrupt the render state mid-cascade; serializing them keeps the UI consistent.

## The hardest part

Performance under GAS's interaction model. Every `onEdit` call pays trigger-startup cost, and naive cell-by-cell rendering of checkbox ranges is brutally slow. Getting from "click, wait three seconds, screen flickers" to a real-time feel required batching all reads/writes into single range operations, minimizing `SpreadsheetApp` calls per trigger, and the cache layer for source data — performance work in an environment that gives you almost no instrumentation to do it with.

## Results

- Invalid work-schedule combinations are impossible: the interface reshapes itself in real time from the source matrix.
- Composing a complex schedule went from hours of copy-paste to minutes of clicking plus a one-click export.
- The interface responds without perceptible load times — uncommon for script-driven spreadsheets and decisive for adoption.
- Faulty exports are recoverable in one click without damaging output files.

## Limitations & what I'd do differently

- `CacheService` has size/TTL limits and the dataset had to be partitioned across multiple cache keys to fit — and a matrix update mid-cache-window serves stale options until expiry.
- The configurator handles composition only — which items and actions apply to an area type. Quantities, frequencies, and commercial calculations were out of scope and handled in a separate step.
- This is the project where Sheets-as-platform reached its ceiling: today, with my fullstack training, this is the first tool I'd rebuild as a small web app (the cascading-filter logic ports directly to React state) — while keeping the source matrix in Sheets where the team maintains it.
