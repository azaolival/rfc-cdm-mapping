# RFC CDM Mapping — Methodology Reference

Source of truth documentation for the discovery and mapping approach. Informs artifact requests, SOW framing, and SME call questions.

---

## Why DDL and WSDL/OpenAPI are the first asks

When a source system has no written specs (RFC's case), the real data dictionary lives in two places: the database schema and the API contract. These are not supporting documents — they ARE the spec.

**DDL gives you:**
- Every entity that exists (Loan, Borrower, Property, Employment, Liability, etc.)
- Primary keys and foreign keys
- Parent/child relationships and cardinality (1:1, 1:many, many:many)
- Data types, constraints, nullability
- Enough to reconstruct the schema from scratch

**WSDL (SOAP) or OpenAPI (REST) gives you:**
- Every business operation the system exposes
- Hierarchical object relationships in requests and responses
- Required vs. optional fields
- Complex object nesting (e.g., Loan contains Borrowers[], Properties[], Liabilities[])
- The logical data model even without direct DB access

**Sample payloads (ULDD, UCD, DU/LPA, URLA, iLAD) give you:**
- Internal field names as they actually populate in practice
- Value-level examples for enum mapping
- Confirmation that the schema and API contract match real loan data

The 80-90% rule: DDL + API contract + 3-5 real loan sample payloads recovers 80-90% of the underlying business data model without needing any other documentation. This is the standard for onboarding to an undocumented mortgage platform.

---

## SOAP vs REST — live question for RFC

RFC's API contract format is UNCONFIRMED. This matters because:

| Format | Ask for |
|---|---|
| SOAP (older pattern, common pre-2015 in-house systems) | WSDL file or WSDL URL (e.g. https://api.company.com/loanService?wsdl) |
| REST (more common post-2015) | OpenAPI/Swagger spec (JSON or YAML) |

RFC's LOS is proprietary and in-house. Systems built pre-2015 in mortgage almost always use SOAP/WSDL. Add this to the first SME call checklist:

> "Is the RFC LOS API SOAP-based (WSDL) or REST-based (OpenAPI/Swagger)?"

The answer changes what you request from RFC engineering and how you process the contract.

---

## Discovery hierarchy (fastest path to source truth)

In priority order when no written spec exists:

1. **DB schema / DDL dump** — INFORMATION_SCHEMA.COLUMNS (SQL Server) or sys.columns or ALL_TAB_COLUMNS (Oracle). FK relationships give parent/child paths. This is the hard blocker.
2. **MISMO XML exports** — RFC cannot originate without these. Each export IS a free internal-to-MISMO crosswalk:
   - ULDD XML: GSE delivery fields (MISMO 3.x)
   - UCD XML: Uniform Closing Dataset (MISMO 3.x, every closed loan)
   - DU/LPA casefile XML: AUS run data (MISMO 3.4, every AUS submission)
   - URLA/1003: Full application dataset (MISMO 3.4)
   - iLAD / servicing boarding file: In-house servicing handoff (MISMO 3.5)
3. **API contract** — WSDL (SOAP) or OpenAPI/Swagger (REST). Reveals logical object model even without DB access.
4. **Live UI proxy trace** — Fiddler / Charles / Burp HAR capture. Last resort, needs screen-share volunteer.
5. **SME time** — LOS builder or contracted dev shop. Undocumented does not mean unknown. 1-2 hrs unblocks days.

---

## What this means for artifact requests and SOW framing

When making the artifact ask to Carrington/RFC stakeholders, the framing is:

- The engagement can recover 80-90% of the source data model without written documentation
- But it cannot start from nothing — the DDL and sample MISMO exports are the minimum viable inputs
- Without the DDL, any mapping produced is an assumption dressed as a spec, which is a risk on a regulated data migration
- The WSDL/OpenAPI contract accelerates the remaining 10-20% and closes gaps where the DB schema alone is ambiguous

This framing supports both the Version A (collaborative) and Version B (blocker escalation) stakeholder emails in `RFC_Encompass_Mapping_KT.md`.

For SOW purposes, scope the deliverable as: field-level mapping complete to the extent that source artifacts are provided. Artifact delivery from RFC is a dependency, not an assumption.

---

## DDL vs DML reference (for stakeholder conversations)

| Type | Purpose | Examples |
|---|---|---|
| DDL | Defines database structure | CREATE, ALTER, DROP, TRUNCATE |
| DML | Manipulates data | SELECT, INSERT, UPDATE, DELETE |

When asking RFC engineering for the DDL, the request is: "the SQL script that would recreate the schema — tables, columns, keys, indexes, constraints — from scratch." Read-only access to a non-prod replica is sufficient. No data is needed, only structure.

---

## SME call checklist (first contact with RFC eng or LOS maintainer)

- [ ] Is "Click n Close" the internal name of the RFC LOS? Is it one system or a front end on something else?
- [ ] What is the source DB engine — SQL Server or Oracle?
- [ ] Is the RFC API SOAP-based (WSDL) or REST-based (OpenAPI/Swagger)?
- [ ] Who owns the DB schema / DDL export?
- [ ] Who owns the MISMO XML export pipeline (secondary/capital markets, closing, underwriting)?
- [ ] Who is the Encompass admin on the Carrington side with access to the field dictionary?
