---
name: fhir-developer
description: >
  Specialized knowledge of HL7 FHIR R4 for healthcare data exchange, with
  deep context for HEDIS, CQL-based digital quality measures (dQMs), Medicare
  Risk Adjustment (HCC/MRA), and clinical rules engine development. Use this
  skill whenever the conversation involves: FHIR resources, resource structures,
  cardinality rules, coding systems (LOINC, SNOMED CT, RxNorm, ICD-10, CPT,
  HCPCS, NDC, CVX), RESTful FHIR API patterns, FHIR validation, CQL measure
  population definitions, dQM data requirements, FHIR-based measure authoring,
  HEDIS measure logic, HCC condition inference, rules engine FHIR compatibility,
  CQL-to-FHIR retrieve mapping, or vendor evaluation of CQL/FHIR engines.
  Always activate when the user asks about HEDIS, Stars, MRA, HCC, CQL, dQM,
  digital quality measures, measure populations, FHIR data infrastructure, or
  clinical rules engine architecture — even if they don't explicitly say "FHIR."
---

# FHIR Developer Skill — HEDIS/CQL/dQM Edition

This skill provides FHIR R4 knowledge scoped specifically for healthcare
quality measurement, CQL-based digital quality measures, HEDIS analytics,
and clinical rules engine evaluation. It extends the baseline FHIR developer
knowledge with measure-specific resource usage, HEDIS-relevant coding systems,
and CQL data requirement patterns.

---

## Core FHIR R4 Knowledge (Baseline)

### Resource Structure Fundamentals
- Every FHIR resource has: `resourceType`, `id`, `meta`, `text`, and domain elements
- `meta.profile` identifies the FHIR profile a resource claims conformance to
- Cardinality notation: `0..1` = optional single, `1..1` = required single, `0..*` = optional repeating, `1..*` = required repeating
- References use `{ "reference": "ResourceType/id" }` format
- Identifiers use `{ "system": "uri", "value": "string" }` — always pair system + value
- CodeableConcept = `{ "coding": [{ "system", "code", "display" }], "text": "..." }`

### RESTful API Patterns
- Base URL format: `[base]/[ResourceType]/[id]`
- Search: `GET [base]/Patient?identifier=...`
- Create: `POST [base]/Patient` with resource body
- Update: `PUT [base]/Patient/[id]`
- Response codes: 200 OK, 201 Created, 400 Bad Request, 404 Not Found, 422 Unprocessable Entity
- Bundles: `type = transaction | batch | searchset | collection | history`
- Bulk FHIR export: `$export` operation on Patient or Group, returns NDJSON

### Validation Patterns
- OperationOutcome is the standard error resource — always return it on 4xx/5xx
- Required fields missing → 422 with `OperationOutcome.issue.severity = error`
- Profile validation: check `meta.profile` then validate against that IG
- Terminology validation: verify codes exist in the declared ValueSet + CodeSystem

---

## HEDIS-Relevant FHIR Resources

These are the resources most commonly referenced in HEDIS measure population
criteria and CQL data requirements.

### Patient
- `birthDate` — used for age-based denominator criteria (e.g., age 18–85)
- `gender` — used in gender-stratified measures
- `address.state` / `address.postalCode` — geographic stratification
- `deceased[x]` — exclusion criteria for deceased members

### Coverage
- `Coverage.status` = `active` — confirms continuous enrollment
- `Coverage.period` — used for continuous enrollment windows (e.g., 11 of 12 months)
- `Coverage.payor` — payer identification for MA vs. commercial
- `Coverage.class[group]` — plan/product line segmentation
- Critical for HEDIS denominator eligibility — most measures require active coverage

### Condition
- `Condition.clinicalStatus` = `active | recurrence | relapse` vs. `inactive | remission | resolved`
- `Condition.verificationStatus` = `confirmed` — HEDIS typically requires confirmed
- `Condition.code` — ICD-10-CM codes via system `http://hl7.org/fhir/sid/icd-10-cm`
- `Condition.onsetDateTime` / `Condition.recordedDate` — for lookback period logic
- `Condition.category` = `problem-list-item` vs. `encounter-diagnosis`

### Encounter
- `Encounter.status` = `finished` — completed visits only for most measures
- `Encounter.class` — `AMB` (ambulatory), `IMP` (inpatient), `EMER` (emergency)
- `Encounter.type` — CPT/HCPCS visit type codes
- `Encounter.period` — start/end datetime for the measurement year window
- `Encounter.participant` — for provider attribution
- `Encounter.diagnosis` — links to Condition; `use` = `chief-complaint | comorbidity | billing`
- `Encounter.hospitalization.dischargeDisposition` — used in readmission measures

### Observation
- `Observation.status` = `final | amended | corrected` (exclude `entered-in-error`)
- `Observation.code` — LOINC codes (system: `http://loinc.org`)
- `Observation.value[x]` — `valueQuantity`, `valueCodeableConcept`, `valueString`
- `Observation.effective[x]` — `effectiveDateTime` or `effectivePeriod`
- `Observation.category` = `laboratory | vital-signs | survey | exam`
- Used for: lab results (A1c, LDL, blood pressure), screenings, PHQ-9 scores

### Procedure
- `Procedure.status` = `completed`
- `Procedure.code` — CPT (system: `http://www.ama-assn.org/go/cpt`) or HCPCS
- `Procedure.performed[x]` — `performedDateTime` or `performedPeriod`
- Primary numerator evidence for many HEDIS measures (screenings, interventions)

### MedicationRequest
- `MedicationRequest.status` = `active | completed`
- `MedicationRequest.medication[x]` — `medicationCodeableConcept` using RxNorm
  (system: `http://www.nlm.nih.gov/research/umls/rxnorm`) or NDC
- `MedicationRequest.dispenseRequest.quantity` + `expectedSupplyDuration` — for PDC calculation
- `MedicationRequest.authoredOn` — prescription date
- Used in PDC (Proportion of Days Covered) measures: CMC, CHL, SAA, etc.

### MedicationDispense
- `MedicationDispense.status` = `completed`
- `MedicationDispense.quantity` + `daysSupply` — **critical for PDC calculation**
- `MedicationDispense.whenHandedOver` — dispense date for supply period calculation
- `MedicationDispense.medication[x]` — same coding as MedicationRequest
- Preferred over MedicationRequest for PDC because it reflects actual dispensing

### DiagnosticReport
- `DiagnosticReport.status` = `final | amended`
- `DiagnosticReport.code` — LOINC panel code
- `DiagnosticReport.result` — references to component Observations
- Used for lab panel results (e.g., lipid panels, metabolic panels)

### Immunization
- `Immunization.status` = `completed`
- `Immunization.vaccineCode` — CVX codes (system: `http://hl7.org/fhir/sid/cvx`)
- `Immunization.occurrenceDateTime` — administration date
- Used in: IMA, Flu, childhood immunization measures

### ServiceRequest
- `ServiceRequest.status` = `active | completed`
- `ServiceRequest.intent` = `order`
- `ServiceRequest.code` — CPT or LOINC order codes
- Used to capture referrals and orders as numerator evidence in some measures

---

## HEDIS-Relevant Coding Systems

| Coding System | FHIR System URI | Used For |
|---|---|---|
| ICD-10-CM | `http://hl7.org/fhir/sid/icd-10-cm` | Diagnoses (Condition, Encounter.diagnosis) |
| ICD-10-PCS | `http://www.cms.gov/Medicare/Coding/ICD10` | Inpatient procedures |
| CPT | `http://www.ama-assn.org/go/cpt` | Outpatient procedures, visits |
| HCPCS | `http://www.cms.gov/Medicare/Coding/HCPCSReleaseCodeSets` | DME, drugs, services |
| LOINC | `http://loinc.org` | Lab orders, results, panels |
| SNOMED CT | `http://snomed.info/sct` | Clinical findings, procedures |
| RxNorm | `http://www.nlm.nih.gov/research/umls/rxnorm` | Medications (clinical drug) |
| NDC | `http://hl7.org/fhir/sid/ndc` | Medications (dispensed product) |
| CVX | `http://hl7.org/fhir/sid/cvx` | Vaccines |
| NPI | `http://hl7.org/fhir/sid/us-npi` | Provider identifiers |
| Revenue Codes | `http://www.nubc.org/RevCode` | UB-04 facility billing |
| DRG | `http://www.cms.gov/Medicare/MCM/Grouper-Software` | MS-DRG inpatient grouping |

**Important:** HEDIS value sets use NCQA-maintained OIDs. When working with
HEDIS CQL measures, value set references look like:
`valueset "Diabetes": 'urn:oid:2.16.840.1.113883.3.464.1003.103.12.1001'`
These must resolve against the VSAC (Value Set Authority Center) or an
equivalent locally-cached terminology service.

---

## CQL-to-FHIR Data Requirement Mapping

CQL measures declare data requirements as retrieve statements. Each retrieve
maps to a specific FHIR resource query.

### Retrieve Pattern
```cql
["Encounter": "Inpatient Encounter"]
```
Maps to FHIR query:
```
GET /Encounter?patient=[id]&type:in=[ValueSet URL]&status=finished
```

### Common CQL Retrieve → FHIR Resource Map

| CQL Data Type | FHIR Resource | Key Filter |
|---|---|---|
| `["Encounter": ...]` | `Encounter` | `type`, `class`, `status`, `period` |
| `["Condition": ...]` | `Condition` | `code`, `clinicalStatus`, `onsetDateTime` |
| `["Observation": ...]` | `Observation` | `code`, `status`, `effectiveDateTime` |
| `["Procedure": ...]` | `Procedure` | `code`, `status`, `performedDateTime` |
| `["MedicationRequest": ...]` | `MedicationRequest` | `medication`, `status`, `authoredOn` |
| `["MedicationDispense": ...]` | `MedicationDispense` | `medication`, `status`, `whenHandedOver` |
| `["Immunization": ...]` | `Immunization` | `vaccineCode`, `status`, `occurrence` |
| `["Coverage": ...]` | `Coverage` | `status`, `period`, `payor` |
| `["Patient"]` | `Patient` | `birthDate`, `gender`, `deceased` |

### Timing Alignment
CQL uses `during` and `overlaps` for interval logic. FHIR queries use:
- `date=ge[start]&date=le[end]` for date range filtering
- Note: CQL interval logic is more expressive than FHIR search — some interval
  operations must be post-filtered in application logic after the FHIR query

---

## HEDIS Measure Population Framework (FHIR context)

HEDIS measures map to FHIR Measure resource populations:

- **Initial Population** → first filter, broad eligibility (age, enrollment, payer)
- **Denominator** → eligible population (may equal IP or be a subset)
- **Denominator Exclusion** → remove members who should be excluded (e.g., hospice)
- **Denominator Exception** → optional exclusions that affect rates differently
- **Numerator** → members who received the target service/outcome
- **Numerator Exclusion** → remove from numerator (rare)

### Key Implementation Notes
- **Continuous enrollment** is a Coverage-based check — not derivable from Encounter data alone
- **Anchor dates** (e.g., index episode start) anchor lookback/lookforward windows
- **Administrative vs. hybrid** data sources: administrative uses claims only;
  hybrid allows supplemental medical record data to fill gaps
- **Custom variants** (e.g., org-specific intervention-friendly measures) may deviate
  from NCQA spec — document deviations explicitly when implementing in CQL

---

## Rules Engine Evaluation Context

When evaluating CQL engine vendors for HEDIS/dQM execution at scale:

### FHIR Infrastructure Questions to Probe
- Does the engine require a live FHIR server, or can it execute against FHIR-shaped data in a database?
- What FHIR version(s) are supported? (R4 is required for NCQA dQMs)
- Does it support FHIR bulk export ($export) for population-level execution?
- How does it handle terminology service calls — cached ValueSets or live VSAC calls?
- What is the FHIR data model requirement — strict resource conformance, or flexible mapping?

### Performance at Scale
- At 14M member scale, FHIR-based measure execution requires efficient bulk data handling
- Watch for engines that make per-member FHIR API calls vs. batch/bulk processing
- Terminology validation overhead (ValueSet membership checks) compounds at scale —
  ask vendors how ValueSets are pre-loaded or cached
- Confirm whether the engine can operate against a FHIR data store replica vs.
  requiring production system access

### Spec-True vs. Customizable Execution
- "No deviation from published CQL" = cannot run org-specific measure variants
- Key flexibility requirement: ability to run both NCQA-spec measures AND
  custom intervention-friendly variants against the same member population
- FHIR profiles can constrain but not extend CQL logic — customization lives in
  the CQL layer, not the FHIR layer

---

## Examples

### Example 1: Check if a member has a completed diabetes encounter in the measurement year
```fhir
GET /Encounter?patient=Patient/12345
  &type:in=https://vsac.nlm.nih.gov/valueset/2.16.840.1.113883.3.464.1003.103.12.1001
  &status=finished
  &date=ge2024-01-01
  &date=le2024-12-31
```

### Example 2: PDC calculation data pull for medication adherence
```fhir
GET /MedicationDispense?patient=Patient/12345
  &medication.code:in=[Statin ValueSet]
  &status=completed
  &whenhandedover=ge2024-01-01
  &whenhandedover=le2024-12-31
```
Then calculate covered days from `quantity.value` / `daysSupply` to get supply periods,
merge overlapping periods, divide total covered days by measurement period days.

### Example 3: HCC-relevant condition lookup (MRA context)
```fhir
GET /Condition?patient=Patient/12345
  &code:in=[HCC-mapped ICD-10 ValueSet]
  &clinical-status=active,recurrence,relapse
  &verification-status=confirmed
  &recorded-date=ge2024-01-01
  &recorded-date=le2024-12-31
```
HCC mapping requires ICD-10-CM → HCC crosswalk applied after retrieval —
FHIR does not natively encode HCC category.

---

## Guidelines

- Always pair `system` + `code` when specifying CodeableConcept — bare codes without
  system URIs are ambiguous and will fail terminology validation
- For HEDIS measure logic, prefer `MedicationDispense` over `MedicationRequest`
  when calculating PDC — dispense reflects actual supply, prescription does not
- When querying for HEDIS denominator eligibility, Coverage period checks are
  mandatory — do not assume enrollment from Encounter presence alone
- NCQA HEDIS value sets must be sourced from VSAC — do not substitute local codes
  without formal equivalence mapping
- For CQL measures, confirm the engine uses FHIR R4 ModelInfo — R3 and R4 have
  different resource structures that will break measure logic if mixed
- Document any deviation from published NCQA CQL spec as a named variant —
  this is essential for audit trail and re-measurement consistency
