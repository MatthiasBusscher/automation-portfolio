# Automation Portfolio

18 production automations I designed and built over 18 months as the sole automation engineer for a multi-entity B2B services group (multiple legal entitie, facility services & security). They cover the full back office: CRM, order-to-cash, accounts payable, ERP integration, HR compliance, and internal tooling.

**Stack:** Google Apps Script · Make.com · Pipedrive · WeFact · Exact Online · Basecone · Dutch government APIs (KvK, Kadaster BAG, PDOK)

> All case studies are anonymized: no client names, IDs, or internal data. The original code is employer-owned and not published here — instead, each case study documents the architecture, the key decisions, and the hardest problems. Generic, reusable patterns extracted from this work are being open-sourced separately.

## Case studies

### 💶 Finance & ERP
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-01"></a>01 | [Recurring Revenue Metrics Engine (ARR/MRR/NRR)](case-studies/01-recurring-metrics-engine.md) | GAS, Sheets | Monthly KPI reporting reduced from hours of manual work to zero |
| <a id="case-06"></a>06 | [Order-to-Cash: Automated Debtor Creation](case-studies/06-order-to-cash-automation.md) | Pipedrive, Make, WeFact | Signed deal → invoice-ready debtor in ~1 second |
| <a id="case-07"></a>07 | [Multi-Entity Cost Center Automation](case-studies/07-multi-entity-cost-centers.md) | Make, Exact Online API | Cost centers created across multiple administrations, automatically |
| <a id="case-14"></a>14 | [Accounts Payable Automation](case-studies/14-accounts-payable-automation.md) | GAS, Basecone | Incoming invoices classified, deduplicated, and routed zero-touch |
| <a id="case-18"></a>18 | [Core Financial ERP Engine: Refactor & Concurrency](case-studies/18-financial-erp-refactor.md) | GAS (9 scripts, 1000+ LOC) | Race-condition data loss eliminated; engine stabilized |

### 🔄 Master data & integration
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-02"></a>02 | [Product Master Data Sync](case-studies/02-product-master-data-sync.md) | GAS, Make, Pipedrive, WeFact | One source of truth for products across 3 systems |
| <a id="case-03"></a>03 | [OAuth2 Token Lifecycle Management](case-studies/03-oauth2-token-management.md) | Make, Exact Online | Financial API integrations stay authenticated, hands-off |
| <a id="case-17"></a>17 | [Sales-to-Operations ETL Pipeline](case-studies/17-sales-to-ops-etl.md) | Pipedrive, Make, Sheets | Won deal → operational planning data, gatekept and transformed |

### 🛡️ Infrastructure & reliability
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-04"></a>04 | [Disaster Recovery: Automated Scenario Backups](case-studies/04-disaster-recovery-backups.md) | Make API, GAS, Drive | All automation logic backed up monthly as JSON blueprints |
| <a id="case-13"></a>13 | [Scanner-to-Cloud Document Routing](case-studies/13-scanner-to-cloud-routing.md) | GAS, Drive API | Physical mail → correct Shared Drive folder, zero-touch |

### 📈 Sales & CRM
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-05"></a>05 | [Lead Routing & CRM Deduplication](case-studies/05-lead-routing-deduplication.md) | Webflow, Make, Pipedrive | Speed-to-lead from hours to seconds, without duplicate records |
| <a id="case-09"></a>09 | [CRM Enrichment: KvK API](case-studies/09-crm-enrichment-kvk.md) | Make, KvK API, Pipedrive | KvK number → verified company profile, automatically |
| <a id="case-10"></a>10 | [Real Estate Enrichment: Kadaster BAG API](case-studies/10-real-estate-enrichment-bag.md) | Make, PDOK, BAG API | Address → registered floor area (m²) for instant quoting |

### 🏢 Internal applications (Google Apps Script)
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-08"></a>08 | [Field Service Reporting Tool](case-studies/08-field-service-reporting.md) | GAS, Sheets, Drive | One-click daily PDF reports for field personnel |
| <a id="case-11"></a>11 | [Enterprise Document Management](case-studies/11-document-management-app.md) | GAS, config-driven | Incoming mail logged, tagged, and chased across 13+ entities |
| <a id="case-12"></a>12 | [GDPR-Compliant HR Document Pipeline](case-studies/12-hr-document-compliance.md) | GAS, Drive | Sensitive HR docs classified and audited, privacy by design |
| <a id="case-15"></a>15 | [Automated Contract Generation](case-studies/15-contract-generation.md) | Forms, GAS, Docs | HR addendum from form input in ~30 seconds, legally consistent |
| <a id="case-16"></a>16 | [Work Schedule Configurator](case-studies/16-work-schedule-configurator.md) | GAS, Sheets UI | Cascading-filter configurator inside a spreadsheet |

## Recurring engineering themes

- **Event-driven over polling** — webhooks fired on precise conditions, saving API quota and cost ([06](#case-06), [09](#case-09), [10](#case-10))
- **Concurrency control** — `LockService` transactional queues against race conditions ([13](#case-13), [16](#case-16), [18](#case-18))
- **Config-driven architecture** — zero data in code; non-technical staff maintain the system via a config sheet ([11](#case-11), [12](#case-12), [14](#case-14))
- **Failure-first design** — fallback routing, retry/catch paths, and audit logs so documents and data can never silently disappear ([13](#case-13), [14](#case-14), [18](#case-18))
- **Privacy by design** — metadata-only notifications and irreversible audit trails for GDPR-sensitive flows ([12](#case-12))
