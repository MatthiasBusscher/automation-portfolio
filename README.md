# Automation Portfolio

A collection of anonymized automation and integration case studies based on professional experience in back-office operations, CRM, finance, document workflows, infrastructure and internal tooling.

The examples describe architecture-level patterns, engineering decisions and lessons learned. They do not include employer-owned code, confidential data, internal screenshots, client names, credentials, workflow exports or identifiable implementation details.

**Typical stack:** workflow automation platforms · Google Workspace automation · CRM/ERP integrations · finance systems · public business-data APIs · custom scripts · GitHub Actions

## Case studies

### 💶 Finance & ERP
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-01"></a>01 | [Recurring Revenue Metrics Engine (ARR/MRR/Growth)](case-studies/01-recurring-metrics-engine.md) | Custom scripts, spreadsheets | Legacy and migrated KPI layouts supported through one calculation path |
| <a id="case-06"></a>06 | [Order-to-Cash: Automated Debtor Creation](case-studies/06-order-to-cash-automation.md) | CRM, workflow automation, finance system | Signed deal data prepared for finance-system intake |
| <a id="case-07"></a>07 | [Multi-Administration Cost Center Automation](case-studies/07-multi-entity-cost-centers.md) | Workflow automation, finance API | Structured records created across connected systems |
| <a id="case-14"></a>14 | [Accounts Payable Automation](case-studies/14-accounts-payable-automation.md) | Custom scripts, document-processing system | Incoming invoices classified, deduplicated and routed with less manual handling |
| <a id="case-18"></a>18 | [Subscription Billing Engine: Migration, Concurrency & Failure Control](case-studies/18-financial-erp-refactor.md) | Custom scripts, locking, billing API | Partial imports and ambiguous billing outcomes moved into explicit recovery paths |

### 🔄 Master data & integration
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-02"></a>02 | [Product Master Data Sync](case-studies/02-product-master-data-sync.md) | Custom scripts, workflow automation, CRM, finance system | Product data synchronized across connected systems |
| <a id="case-03"></a>03 | [OAuth2 Token Lifecycle Management](case-studies/03-oauth2-token-management.md) | Workflow automation, finance API | Financial API authentication maintained through scheduled token refresh |
| <a id="case-17"></a>17 | [Sales-to-Operations ETL Pipeline](case-studies/17-sales-to-ops-etl.md) | CRM, workflow automation, spreadsheet-based staging | Won-deal data transformed for operational planning |

### 🛡️ Infrastructure, CI/CD & reliability
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-04"></a>04 | [Disaster Recovery: Automated Scenario Backups](case-studies/04-disaster-recovery-backups.md) | Workflow automation API, custom scripts, cloud storage | Automation configuration backed up for recovery |
| <a id="case-13"></a>13 | [Scanner-to-Cloud Document Routing](case-studies/13-scanner-to-cloud-routing.md) | Custom scripts, cloud storage API | Incoming documents routed to the correct secure location |
| <a id="case-19"></a>19 | [Reusable GAS Deployment Framework](case-studies/19-reusable-gas-deployment-framework.md) | GitHub Actions, Git, script deployment tooling | Version-controlled deployments standardized with reusable automation |

### 📈 Sales & CRM
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-05"></a>05 | [Multi-Form Lead Routing, Qualification & CRM Deduplication](case-studies/05-lead-routing-deduplication.md) | Web forms, workflow automation, CRM, email platform | Inbound form submissions deduplicated, qualified and segmented through one workflow |
| <a id="case-09"></a>09 | [CRM Enrichment: KvK API](case-studies/09-crm-enrichment-kvk.md) | Workflow automation, public business-data API, CRM | Business identifier used for company-profile enrichment |
| <a id="case-10"></a>10 | [Real Estate Enrichment: Kadaster BAG API](case-studies/10-real-estate-enrichment-bag.md) | Workflow automation, public property-data API | Address data enriched for operational estimation |

### 🏢 Internal applications (Google Apps Script)
| # | Case study | Stack | Headline result |
|---|-----------|-------|-----------------|
| <a id="case-08"></a>08 | [Field Service Reporting Tool](case-studies/08-field-service-reporting.md) | Custom scripts, spreadsheets, cloud storage | Daily reports generated from structured field input |
| <a id="case-11"></a>11 | [Enterprise Document Management](case-studies/11-document-management-app.md) | Custom scripts, config-driven spreadsheet | Incoming mail logged, tagged and followed up from a central view |
| <a id="case-12"></a>12 | [Privacy-Conscious HR Document Workflow](case-studies/12-hr-document-compliance.md) | Custom scripts, cloud storage | Privacy-sensitive HR documents classified and reviewable |
| <a id="case-15"></a>15 | [Automated Contract Generation](case-studies/15-contract-generation.md) | Forms, custom scripts, document automation | HR document generated from structured form input |
| <a id="case-16"></a>16 | [Work Schedule Configurator](case-studies/16-work-schedule-configurator.md) | Custom scripts, spreadsheet UI | Spreadsheet-based configurator built with cascading filters |

## Recurring engineering themes

- **Event-driven design** — workflows trigger from meaningful business events rather than relying only on manual checks.
- **Concurrency control** — locking and queueing patterns reduce race conditions in shared systems.
- **Idempotent recovery** — interrupted local processing is repaired from known state without duplicating complete records.
- **Schema-aware migration** — canonical fields, aliases and preflight checks keep workbook changes from silently shifting business logic.
- **Config-driven architecture** — operational settings are moved out of code where appropriate, so non-technical users can maintain parts of the process safely.
- **Failure-aware design** — fallback paths, retry boundaries and audit logs make issues easier to detect and recover from.
- **Privacy-conscious workflows** — sensitive content is kept out of broad notification channels where possible.
