# RFC (Reliance) LOS to Encompass Data Mapping — Engagement KT

> Knowledge transfer doc for the Claude Code project. Drop this in the repo so CC has the full context, methodology, and current state of the mapping work.

## House style
- Never use em-dashes. Use commas, parentheses, or separate sentences.
- Hunt and remove AI footprints (filler openers, buzzword stacking, overly smooth cadence) before any writing goes out the door. It should read like a real person wrote it.
- AzaRiz lens applies: Aza brings the domain judgment and people-reading, CC brings speed and execution. Aza's voice stays in the final work.

---

## 1. Engagement context

Aza is engaged as a consultant by **Carrington** to map the loan-data layer of **Reliance First Capital (RFC)** to **Encompass** as the go-forward system of record. Carrington is acquiring Reliance (deal announced Oct 31 2025, expected to close Q1 2026), adding a direct-to-consumer origination channel to Carrington's existing retail recapture, wholesale, and correspondent businesses.

The job: map RFC's source-of-truth loan data, stored in their proprietary LOS, to Encompass field IDs using standard MISMO as the common language.

## 2. Critical naming note: "Click n Close"

There are two unrelated things that share this name. Do not conflate them.

- **Click n' Close, Inc.** is a separate Texas mortgage lender (the rebrand of Mid America Mortgage, owned by Jeff Bode). It is the nation's largest Section 184 (Native American) lender via its 1st Tribal Lending division. It is a company, not a platform, not a POS, not an LOS. Servicing runs on Sagent (moving to Dara as of Feb 2026). This is a red herring for our purposes.
- **"Click n Close" as RFC's internal system name** is the working hypothesis. RFC built a proprietary in-house LOS, and an in-house system almost always carries an internal product name that never shows up in vendor press. So the name Aza was given may be RFC's internal label for their own LOS. **Unconfirmed.** First question for any RFC SME call: is "Click n Close" the name of your origination platform, and is it one homegrown system or a homegrown front end on top of something else?

## 3. What RFC actually is, and how they originate

- Direct-to-consumer, call-center driven. Six origination centers, ~100+ licensed "Mortgage Analysts" (their internal title for LOs) plus ~100+ support staff. ~$1B/year, $13B+ originated since 2008.
- Lead-gen heavy, historically a top-ten lender on the **LendingTree** network. Purchased/aggregator leads worked by phone, plus a realtor-partner purchase program.
- Products: FHA, VA, USDA, conventional, jumbo, Non-QM. Refi and purchase. Second-lien products exist.
- **Tech: proprietary, in-house LOS.** No name-brand LOS (not Encompass, not Calyx, not BytePro). They service in-house, which means there is a servicing boarding handoff in play (relevant to MISMO 3.5 / iLAD).

## 4. The core problem

The target (Encompass) is documented. The source (RFC's proprietary LOS) has **no tech specs and no MISMO specifications written down.** That is normal for an in-house system. We are not mapping documented-to-documented. We are reverse-engineering an undocumented source against a documented target.

The fix: stop waiting for specs that were never written. The spec already exists in three places, it just has to be extracted.

## 5. Methodology: MISMO as the pivot

Map **source field -> MISMO XPath -> Encompass field ID** once. That single row becomes the migration spec. MISMO is the shared language between the two systems.

The reason this works: RFC cannot originate without producing standardized MISMO datasets. Every one of those exports is already a pre-built crosswalk from their internal fields to a known MISMO container. Triangulate the exports against the database schema and the UI, and the mapping reveals itself.

### Discovery hierarchy (fastest path to truth)
1. **Database catalog** = the real data dictionary. `INFORMATION_SCHEMA.COLUMNS` (SQL Server) / `sys.columns` / `ALL_TAB_COLUMNS` (Oracle). FK relationships give the parent/child paths on the source side. When no spec exists, you generate one from the catalog.
2. **MISMO exports** = internal-field-to-XPath crosswalk, free of charge:
   - ULDD XML (GSE delivery) if they sell to Fannie/Freddie
   - UCD XML (closing dataset, every closed loan)
   - DU and/or LPA casefile XML (ULAD / MISMO 3.4, every AUS run)
   - URLA / 1003 (ULAD, MISMO 3.4)
   - Servicing boarding / transfer file (iLAD, the MISMO 3.5 touchpoint)
   - Doc-engine payload (DocMagic / IDS / Docutech)
3. **API contract** (OpenAPI/Swagger or WSDL) if one exists. If not, proxy a live loan session (Fiddler / Charles / Burp) to reverse-map UI field -> JSON path -> column.
4. **The human who built or maintains it.** Undocumented does not mean unknown. Internal eng or a contracted dev shop has it in their head.

### MISMO version notes
- URLA/ULAD and AUS casefiles ride MISMO **v3.4**.
- ULDD and UCD ride MISMO **3.x**.
- Servicing boarding / iLAD is the **3.5** touchpoint. RFC services in-house, so this handoff is a distinct mapping from origination.

## 6. The crosswalk

Working file is `RFC_Encompass_MISMO_Crosswalk.xlsx` (for Excel review) with CSV exports for in-repo editing: `RFC_crosswalk.csv`, `RFC_enums.csv`, `RFC_inventory.csv`.

### Crosswalk columns
`Loan Data Domain | Business Field Name | RFC Source Table.Column | Src Datatype | Src Sample Value | MISMO Container/XPath | MISMO Ver | Encompass Field ID | Encompass Field Description | Transform/Business Rule | Discovery/Lineage Source | Status | Notes`

Workflow per row: fill source columns from the schema + exports, validate the Encompass field ID against the live field dictionary, capture any value-level code mapping in the enum table, then move Status from `Unmapped` -> `Proposed` -> `Validated`.

> The pre-filled Encompass field IDs are common anchor fields shown to demonstrate the pattern. Every ID MUST be confirmed against the actual Encompass field dictionary for this instance. Source columns are all TBD pending the schema dump.

### Pre-seeded anchor rows

| Domain | Business Field | MISMO XPath (abbrev) | Ver | Encompass ID | Transform note |
|---|---|---|---|---|---|
| Borrower | First Name | .../INDIVIDUAL/NAME/FirstName | 3.4 | 4000 | Trim whitespace |
| Borrower | Last Name | .../INDIVIDUAL/NAME/LastName | 3.4 | 4002 | Direct |
| Borrower | SSN | .../TAXPAYER_IDENTIFIER/TaxpayerIdentifierValue | 3.4 | 65 | Strip dashes, PII mask in non-prod |
| Borrower | Email | .../CONTACT_POINT_EMAIL/ContactPointEmailValue | 3.4 | 1240 | Direct |
| Borrower | Phone | .../CONTACT_POINT_TELEPHONE/ContactPointTelephoneValue | 3.4 | confirm | Normalize 10-digit, map use-type |
| Borrower | Co-Borrower link | .../RELATIONSHIPS/RELATIONSHIP | 3.4 | role-based | Watch multi-borrower cardinality |
| Property | Street Address | .../SUBJECT_PROPERTY/ADDRESS/AddressLineText | 3.4 | 11 | Concat house no + street if split |
| Property | City | .../SUBJECT_PROPERTY/ADDRESS/CityName | 3.4 | 12 | Direct |
| Property | State | .../SUBJECT_PROPERTY/ADDRESS/StateCode | 3.4 | 14 | Confirm 2-letter vs full |
| Property | Zip | .../SUBJECT_PROPERTY/ADDRESS/PostalCode | 3.4 | 15 | Split Zip / Zip+4 if needed |
| Property | Appraised Value | .../PROPERTY_VALUATION/.../PropertyValuationAmount | 3.4 | 1041 | Pick valuation type = Appraisal |
| Property | Purchase Price | .../SALES_CONTRACT/SalesContractAmount | 3.4 | 136/356 | Purchase only, null on refi |
| Loan Terms | Base Loan Amount | .../TERMS_OF_LOAN/BaseLoanAmount | 3.4 | 1109/2 | Distinguish base vs total |
| Loan Terms | Total Loan Amount | .../LOAN_DETAIL/TotalLoanAmount | 3.4 | 1109 | Base + financed fees |
| Loan Terms | Note Rate | .../TERMS_OF_LOAN/NoteRatePercent | 3.4 | 3 | Confirm decimal vs basis-pt |
| Loan Terms | Loan Term (months) | .../MATURITY_RULE/LoanMaturityPeriodCount | 3.4 | 4 | Confirm months vs years |
| Loan Terms | Lien Position | .../LOAN_DETAIL/LienPriorityType | 3.4 | 1811 | Enum, see enum table |
| Loan Terms | Loan Type | .../TERMS_OF_LOAN/MortgageType | 3.4 | 1172 | Enum, see enum table |
| Loan Terms | Loan Purpose | .../LOAN_DETAIL/LoanPurposeType | 3.4 | 19 | Enum, see enum table |
| Loan Terms | Amortization Type | .../AMORTIZATION_RULE/AmortizationType | 3.4 | 608 | ARM adds margin/index/caps rows |
| Dates | Application Date | .../APPLICATION/ApplicationReceivedDate | 3.4 | 745 | Confirm tz / date-only |
| Dates | Closing/Note Date | .../EXECUTION/ExecutionDate | 3.4 | 748 | Closed loans only |
| Income | Total Monthly Income | .../CURRENT_INCOME_ITEM/CurrentIncomeMonthlyTotalAmount | 3.4 | 736 | Reconcile to AUS qualifying income |
| Underwriting | Back-end DTI | .../QUALIFICATION/TotalDebtExpenseRatioPercent | 3.4 | 742 | Recompute vs store-and-carry |
| Underwriting | Rep Credit Score | .../CREDIT_SCORE/CreditScoreValue | 3.4 | varies | Map rep-score rule (mid/lower-of) |
| Servicing | Servicing/Investor IDs | iLAD .../LOAN_IDENTIFIER | 3.5 | svc fields | In-house servicing boarding handoff |

### Enum mappings (value-level, all source codes TBD)

| Enum Set | RFC Meaning | MISMO Enum | Encompass Value | Note |
|---|---|---|---|---|
| Loan Type | Conventional | Conventional | Conventional | |
| Loan Type | FHA | FHA | FHA | |
| Loan Type | VA | VA | VA | |
| Loan Type | USDA/Rural | USDARural | USDA-RuralHousing | |
| Loan Purpose | Purchase | Purchase | Purchase | |
| Loan Purpose | Rate/Term Refi | Refinance | NoCash-Out Refinance | Confirm refi sub-type split |
| Loan Purpose | Cash-Out Refi | Refinance | Cash-Out Refinance | May be separate source flag |
| Lien Position | First | FirstLien | FirstLien | |
| Lien Position | Second | SecondLien | SubordinateLien | RFC does second-lien products |
| Amortization | Fixed | Fixed | Fixed | |
| Amortization | ARM | AdjustableRate | AdjustableRate | Triggers ARM sub-mapping |
| Occupancy | Primary | PrimaryResidence | PrimaryResidence | |
| Occupancy | Second Home | SecondHome | SecondHome | |
| Occupancy | Investment | Investor | InvestmentProperty | |
| Property Type | SFR | Detached | SingleFamily | Map full property-type domain |

## 7. Source artifact inventory (discovery checklist)

Collect these first. Items 1 and 2 are hard blockers.

1. **DB schema / DDL dump** (non-prod replica, read-only). The real data dictionary. Owner: RFC eng/DBA.
2. **ULDD XML** sample, 3-5 closed loans. Owner: RFC secondary/capital markets. Only if they sell to GSEs.
3. **UCD XML** sample, 3-5 closed loans. Owner: RFC closing/post-closing. Every closed loan has one.
4. **DU and/or LPA casefile XML.** Owner: RFC underwriting. Every AUS run produces one.
5. **URLA / 1003 export.** Owner: RFC origination.
6. **Servicing boarding / transfer file** (iLAD, 3.5). Owner: RFC servicing.
7. **Doc-engine payload** (identify DocMagic/IDS/Docutech first). Owner: RFC closing.
8. **API contract** (OpenAPI/Swagger or WSDL). Owner: RFC eng. Ask before resorting to a proxy trace.
9. **Live-loan UI session trace** (Fiddler/Charles/Burp HAR). Last resort, needs a screen-share volunteer.
10. **SME time** with the LOS builder/maintainer, 1-2 hrs. Internal eng or contracted dev shop.
11. **Encompass field dictionary** for the target instance. Owner: Carrington Encompass admin. Validate every ID against this.
12. **ERD** if one exists. Likely doesn't, derive from FKs.

## 8. Open questions / next steps

- Confirm whether "Click n Close" is RFC's internal LOS name, and whether it is one system or a front end on something else. This changes how many source schemas we map.
- Confirm the source DB engine (SQL Server vs Oracle) to pick the right catalog queries.
- Identify the doc engine.
- Decide rep-credit-score selection rule explicitly before mapping.
- ARM sub-mapping block (margin, index, caps, first-adjustment) not yet built. Add when an ARM loan sample is in hand.
- iLAD 3.5 servicing field block not yet expanded. Add once the boarding file sample arrives.

## 9. Stakeholder ask (email, two versions)

### Version A — Collaborative unblock
Subject: Reliance LOS mapping: access and artifacts I need to start

Hi [Name],

Quick update on the Reliance-to-Encompass data mapping. The target side is straightforward since Encompass is a documented system of record. The gap is the source: Reliance's LOS is proprietary and there don't appear to be tech specs or MISMO documentation written down for it. That's normal for an in-house system, and I can work around it, but I need a few things to reverse-engineer the source of truth efficiently rather than guess at it.

Here's what unblocks me, in priority order:

1. Read-only access to a non-prod copy of the Reliance LOS database, plus a schema/DDL dump. The schema is effectively the data dictionary that was never written down. Read-only is fine.
2. Sample full-loan exports, 3 to 5 real loans (de-identified is fine for non-prod), in whatever standardized formats the system already produces: ULDD, UCD, DU and/or LPA casefile XML, URLA/1003, and a servicing boarding/transfer file. Each of these is already a mapping from their internal fields to a known MISMO container, so they do most of the work for me.
3. If the LOS has an internal API, the OpenAPI/Swagger spec or WSDL. If not, one short screen-share with a Reliance user clicking through a full loan so I can trace the field-to-data path.
4. One to two hours with whoever built or currently maintains the LOS, internal or a contracted dev shop. Undocumented doesn't mean unknown, and that conversation will save days.
5. Access to the Encompass field dictionary for the target instance so I can validate every field ID I map to.

If we can line up 1, 2 and 5 first, I can start producing the crosswalk immediately and refine from there. Happy to hop on a call to scope access. Who owns the Reliance database and Encompass admin on your side?

Thanks,
Aza

### Version B — Flag as blocker
Subject: Dependency to start Reliance LOS mapping - source specs don't exist

Hi [Name],

Before I get too far, I want to flag a dependency clearly so we set expectations right.

The engagement assumes I can map Reliance's loan data to the Encompass system of record. Encompass is documented, so the target is fine. The issue is the source: Reliance's LOS is proprietary and there are no tech specs or MISMO specifications to work from. They don't exist. I can reverse-engineer the source of truth, but not from nothing, and not by guessing at field meanings on a system this regulated.

To keep this on schedule, I need the following. Items 1 and 2 are hard blockers:

1. Read-only access to a non-prod Reliance LOS database and a schema/DDL dump. This is the actual data dictionary.
2. 3 to 5 sample full-loan exports in the standardized formats the system already emits (ULDD, UCD, DU/LPA XML, URLA, servicing boarding file). These are existing internal-to-MISMO crosswalks and are the fastest path to an accurate mapping.
3. The Encompass field dictionary for the target instance, to validate the mapping.
4. 1 to 2 hours with the LOS builder/maintainer (internal or contracted dev shop).

Without 1 and 2, the deliverable would be assumptions dressed up as a spec, which is a risk none of us wants on a data migration. Can you point me to the database owner and the Encompass admin so I can get access requests moving today?

Thanks,
Aza
