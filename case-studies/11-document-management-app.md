# Enterprise Document Management App (Config-Driven GAS)

> **Context** Umbrella organization, a multi-entity group · daily inflow of critical mail and tax authority correspondence via Drive
> **Stack** Google Apps Script · Google Sheets (logbook + config) · Drive · Gmail (HTML digests)
> **Category** Internal application development

## The problem

Critical post for multiple entities landed across dozens of Drive folders with no central overview: who needs to read which letter, what follow-up actions exist, and were they done? Beyond the workflow problem sat an architectural one: entity names change, new entities get founded, folder structures move. If all of that is hardcoded, every organizational change needs a developer — making the system itself the bottleneck it was meant to remove.

## Architecture

```mermaid
flowchart LR
    subgraph Ingest
    CRON5["5-min trigger"] --> PROC["processOtherPost:<br/>scan entity + tax folders"]
    PROC --> REGEX["parseFromFileName_:<br/>entity recognition (regex)"]
    REGEX --> LOG[("Central logbook<br/>(Sheets)")]
    end
    subgraph Config
    CFG[("Configuration tab:<br/>folder IDs, entities,<br/>email settings")] --> JSON["Parsed to JSON object<br/>cached via CacheService"]
    JSON -.-> PROC
    JSON -.-> DIGEST
    end
    subgraph Workflow
    LOG --> EDIT["onEdit: tag colleagues,<br/>mark as read"]
    EDIT --> JOBS["Background job system<br/>collects tags"]
    JOBS --> DIGEST["Scheduled HTML digests<br/>(06:00 follow-ups, 11:00 new post)"]
    end
```

A file processor sweeps the entity and tax-authority folders every 5 minutes, recognizes the owning entity from filename conventions, and writes each document into a central logbook. Staff work *in* the logbook — tagging colleagues, marking items read — and a job system batches those events into scheduled, personalized HTML digests. Critically, the code contains **no data**: every folder ID, entity name, and send time lives in a Configuration tab the script parses (and caches) at runtime.

## Key decisions & trade-offs

- **Config-driven over hardcoded — the central decision.** Administration staff add an entity or change a digest time by editing a spreadsheet row, never touching code. Cost: the script must validate its own config defensively (a typo'd folder ID is now runtime data, not a compile-time error). Worth it: the developer bottleneck is significantly reduced — adding a new entity takes a few targeted code changes rather than an architectural overhaul, with the config handling the rest. The system has required no structural changes since launch despite multiple reorganizations.
- **Scheduled digests instead of instant notifications.** A mail per document would train everyone to ignore notifications within a week. Batched digests at deliberate times (06:00 follow-up overview to start the day, 11:00 new-post overview) respect attention as the scarce resource.
- **`CacheService` for the config.** The config parse is re-needed by every trigger run; caching the parsed JSON keeps the 5-minute processor and the `onEdit` handlers fast instead of re-reading the sheet each invocation.
- **Native data validation tied to config.** Dropdowns in the logbook sync from the configuration, so invalid entities or statuses can't be entered — pollution prevented at the cell level rather than cleaned up afterwards.

## The hardest part

Designing the config schema so non-technical staff could *safely* hold the keys. The naive version — "put folder IDs in a sheet" — fails the first time someone pastes a URL instead of an ID or leaves a row half-filled. The robust version validates every row, fails loudly with a readable error pointing at the offending cell, and treats the cache correctly when config changes mid-day. Making a system *less* fragile while making it *more* editable by laypeople is the real trick.

## Results

- Incoming documents appear in the right logbook within 5 minutes, with correct file links and entity attribution — zero manual entry.
- Config changes (folders, schedule times, recipients) are made by administration staff in the config tab without developer involvement; no code changes to the engine since launch.
- Follow-up on critical government correspondence is structurally faster: tagged actions surface in the morning digest until handled.
- Invalid logbook entries are impossible by construction (config-synced validation).

## Limitations & what I'd do differently

- Entity recognition depends on filename conventions; misnamed files need manual correction in the logbook. An OCR/content-based classifier would remove the convention dependency — at significant complexity cost.
- The 5-minute sweep is polling; fine at this volume, but Drive push notifications (via a small web app endpoint) would be the cleaner trigger at larger scale.
- GAS quotas (trigger runtime, email/day) bound the design; the architecture works *with* them, but a much larger organization would outgrow the platform.
