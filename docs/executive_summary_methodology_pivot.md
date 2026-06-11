# RFC to Encompass Data Mapping: Executive Summary and Methodology Recommendation

Prepared by: Aza Olivar, BSA Consultant
Date: June 11, 2026
Audience: Carrington Mortgage Leadership

---

## Situation

Carrington engaged a BSA consultant to map loan data from Reliance First Capital's (RFC) proprietary LOS to Encompass as the go-forward system of record following the acquisition. The intent is to produce a field-level mapping specification that governs data migration, integration, and system alignment across the mortgage loan lifecycle.

After reviewing the current engagement structure, the available artifacts, and the nature of RFC's technology stack, I need to surface a critical finding before further mapping work proceeds.

---

## The Core Finding

The current approach frames this as a direct database-to-database mapping: match RFC SQL columns to Carrington Encompass fields. That framing is creating unnecessary difficulty and will produce lower-quality results than an alternative approach already available to us. Here is why.

RFC's LOS is a proprietary, in-house system built and maintained by their own engineering team. It has no vendor documentation, no published MISMO specifications, and no written data dictionary. The schemas we are working from on the RFC side are informal, inconsistently documented, and have not been validated against what the system actually stores and exchanges in production.

A direct column-to-column mapping under these conditions means we are making educated guesses dressed as specifications. In a regulated data migration context, that is a meaningful risk.

---

## A Better Path Already Exists

RFC cannot originate, close, or deliver loans without producing industry-standard MISMO datasets. This is not optional. Every AUS submission, every GSE delivery, every closing, and every URLA generates a MISMO XML export that maps RFC's internal fields to a known, auditable MISMO container. Those exports exist today, sitting in RFC's systems, and they are already doing the translation work we are trying to recreate manually.

The correct architecture for this engagement is:

RFC source field → MISMO XPath → Encompass Field ID

This is a three-column crosswalk that uses MISMO as the shared language between the two systems. It works because:

Encompass is MISMO-native. Every Encompass field ID has a corresponding MISMO path. That mapping is already documented and published.

RFC's MISMO exports are the spec that was never written. The UCD XML (every closed loan), the DU/LPA casefile (every AUS run), the ULDD (every GSE delivery), and the URLA export collectively reveal how RFC's internal data maps to MISMO containers. We extract the crosswalk from the exports rather than guessing at it from inconsistent spreadsheets.

The result is a single mapping row: RFC source field, MISMO XPath, Encompass Field ID. Each row is auditable, industry-standard, and defensible. It is also faster to produce because we are working with documented systems on both ends once the MISMO pivot is in place.

---

## What Is Blocking Progress Right Now

Two categories of blockers are preventing high-confidence mapping work, regardless of methodology.

Category 1: Source artifacts not yet provided

To execute the MISMO-pivot approach, we need the following from RFC. Items 1 and 2 are hard blockers. Nothing maps cleanly without them.

1. Read-only access to the RFC LOS database with a schema or DDL export. This is the actual data dictionary. RFC engineering or their DBA team owns this.
2. Sample MISMO XML exports from 3 to 5 real closed loans, de-identified if needed. This means UCD, ULDD or DU/LPA casefile, and URLA. These files exist and are produced routinely. RFC closing, underwriting, and secondary/capital markets teams own them.
3. Identification of RFC's API contract, either a WSDL if the system is SOAP-based or an OpenAPI/Swagger spec if it is REST-based. RFC engineering owns this.
4. One to two hours with whoever built or currently maintains the RFC LOS, whether internal or a contracted development shop.

Category 2: Destination validation not yet established

5. The Encompass field dictionary for the Carrington instance. Every Encompass field ID in the mapping must be validated against the actual target instance, not against generic documentation. Carrington's Encompass administrator owns this.

Without items 1 and 2, any mapping produced carries significant confidence risk. The mapping work we can do today is a starting point, not a finished specification.

---

## What Has Been Done and What It Shows

A working crosswalk has been pre-seeded with 26 anchor field rows covering Borrower, Property, Loan Terms, Dates, Income, Underwriting, and Servicing domains. All RFC source columns are currently marked as pending because the source artifacts listed above have not been received. The MISMO XPath and Encompass Field ID columns are pre-populated based on industry-standard patterns and are ready for validation.

Additionally, a shared mapping tool is in development that will allow Carrington and RFC stakeholders to view the mapping state, review confidence scores, and track blocked items in a shared environment, without requiring either side to access the other's internal project management tools. This addresses a real workflow gap: RFC cannot access Carrington's ADO or JIRA, and the current email and spreadsheet exchange is producing slow and difficult-to-audit results.

---

## What I Am Asking Leadership to Approve

Three things.

First, endorse the MISMO-pivot methodology (RFC source field → MISMO XPath → Encompass Field ID) as the official mapping approach for this engagement. This is the right architecture for an undocumented source system and a MISMO-native target. It produces a more defensible and auditable deliverable.

Second, assign artifact owners and set a delivery deadline for the five items listed above. The DDL/schema dump and MISMO XML samples from RFC are the critical path. Without them, confidence in the output cannot exceed medium for most field rows.

Third, approve the shared mapping and tracking tool as the official workflow for this engagement. Replacing email and spreadsheet exchange with a shared, always-available environment will accelerate review cycles and create a clear audit trail of mapping decisions and open items.

---

## Risk of the Status Quo

If the engagement continues on the current direct DB-to-DB path without the source artifacts and without methodology alignment, the likely outcome is a mapping specification that takes significantly longer to produce, carries lower confidence on most rows, and is harder to validate when it is handed off to the Carrington development team to execute. Any downstream errors in a regulated data migration carry operational, compliance, and audit risk.

Catching and correcting the approach now, before significant additional work is invested, is the right call.

---

Prepared to discuss and answer questions at any time.

Aza Olivar
BSA Consultant, RFC to Encompass Data Mapping
