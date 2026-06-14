# Field Service Reporting Tool (One-Click Daily PDF Reports)

> **Context** Security staff at a client site · daily shift reports to the end client
> **Stack** Google Apps Script · Google Sheets (UI) · Drive export API · Gmail (HTML mail)
> **Category** Custom internal tools

## The problem

Security guards ended every shift with the same administrative ritual: type out routine tasks by hand, wrestle a spreadsheet into a presentable PDF, and email it to the right stakeholders. Reports were inconsistent (forgotten tasks, typos), formatting broke regularly, and mails occasionally went to the wrong addresses. The end client's daily impression of the service was formed by its sloppiest artifact. The goal: a one-click experience for users with zero IT affinity.

## Architecture

```mermaid
flowchart LR
    GUARD["Guard selects date"] --> EDIT["onEdit trigger"]
    EDIT --> DAY["Detect weekday"]
    DAY --> FILL["Auto-fill roster +<br/>standard tasks from<br/>hidden data sheet"]
    GUARD2["Guard adds remarks,<br/>clicks 'Send PDF' (custom menu)"] --> EXPORT["Drive export API:<br/>render sheet as scaled PDF"]
    EXPORT --> MAIL["HTML email<br/>(logo + signature)<br/>+ PDF attachment"]
    MAIL --> STAKE["Hardcoded recipients<br/>(TO / CC)"]
```

A Google Sheet doubles as the application UI. Selecting the date fires an `onEdit` trigger that recognizes the weekday and pre-loads that day's roster and standard tasks from a hidden data sheet. A custom menu button renders the sheet as a properly scaled PDF via the Drive export API and sends it inside a branded HTML email to preset stakeholders.

## Key decisions & trade-offs

- **A spreadsheet as the app.** Field staff already knew Sheets; a web app or mobile app would have meant training, login management, and development time for what is fundamentally a daily form. The trade-off — Sheets' limited UI control — was acceptable for a single-screen workflow.
- **Day-specific auto-fill from a hidden data sheet.** Routine tasks differ per weekday; encoding them as data (not code) means a supervisor can adjust the standard task list without touching the script.
- **Hardcoded recipients, deliberately.** The one place where flexibility was *removed*: guards cannot change who receives the report. Misdirected reports to clients were a real incident class; eliminating the input eliminates the incident.
- **Drive export API over a Docs/PDF library.** Exporting the sheet itself (with tuned scaling parameters) keeps the report layout maintainable in the sheet, visible to the people who use it — no separate template to keep in sync.

## The hardest part

Getting the PDF rendering production-quality. The Drive export endpoint takes a set of poorly documented URL parameters (size, scale, gridlines, print area), and the defaults produce exactly the broken layouts the team suffered from manually. Finding the parameter combination that rendered the report correctly regardless of how much text a guard entered took sustained trial and error.

## Results

- The entire end-of-shift process is one date selection plus one button click; no IT knowledge, downloads, or manual formatting involved.
- Writing out routine tasks eliminated via day-specific auto-fill — and "forgotten" standard tasks no longer happen.
- The client receives a consistently formatted, branded PDF in a professional HTML email every day — feedback confirmed they were happy to receive a clear, consistent report rather than the irregular output they'd seen before.
- Misdirected reports: zero, by construction.

## Limitations & what I'd do differently

- Recipients being hardcoded means a personnel change on the client side requires a script edit — the right fix is a protected config range (the pattern used in the [document management app](11-document-management-app.md)), keeping the no-guard-access property while making changes self-service for supervisors.
- One sheet serves one site; only one location was in production, though the architecture (data sheet + script) is designed to replicate easily to additional sites by copying the template.
- No offline path: a guard without connectivity at shift end can't file the report from the field.
