# ABFM Skills Assessment

## Tools & Approach

I used Claude as a coding agent throughout this assessment to accelerate 
implementation. All analytical decisions — which questions to prioritize, 
which methods to apply, how to interpret findings — were my own. I reviewed, 
tested, and modified all generated code before including it, and I can speak 
to every result here.

## Data Overview

Nine CSV files of synthetic EHR (Synthea-style) patient data, linked by patient, encounter, and payer IDs.

| Table | Rows | Cols | Description |
|---|---:|---:|---|
| `patients` | 10,000 | 28 | Demographics, location, income, healthcare expenses |
| `encounters` | 1,809,356 | 15 | Clinical encounters with cost, payer, provider, reason |
| `observations` | 9,020,852 | 9 | Lab results, vitals, survey scores per encounter |
| `conditions` | 1,032,326 | 7 | Diagnoses with start/stop dates |
| `medications` | 1,460,996 | 13 | Prescriptions with cost, payer coverage, dispenses |
| `allergies` | 21,045 | 15 | Allergy records with reactions and severity |
| `payers` | 40 | 22 | Insurance payer summary stats |
| `payer_transitions` | 1,168,680 | 8 | Patient insurance enrollment history |
| `US_zip_elevation` | 41,482 | 2 | Reference: mean elevation by ZIP code |

## Entity Relationships

```
                         ┌──────────────┐
                         │   patients   │
                         │  (Id = PK)   │
                         └──────┬───────┘
                                │ PATIENT
            ┌───────────┬───────┼────────┬──────────────┬───────────────┐
            │           │       │        │              │               │
            ▼           ▼       ▼        ▼              ▼               ▼
     ┌────────────┐ ┌────────┐ ┌──────────┐ ┌────────────┐ ┌─────────────────┐
     │ encounters │ │allergies│ │conditions│ │medications │ │payer_transitions│
     │ (Id = PK)  │ │        │ │          │ │            │ │                 │
     └──────┬─────┘ └───┬────┘ └────┬─────┘ └─────┬──────┘ └────────┬────────┘
            │           │           │              │                 │
            │      ENCOUNTER   ENCOUNTER      ENCOUNTER           PAYER
            │◄──────────┘           │              │                 │
            │◄──────────────────────┘              │                 ▼
            │◄─────────────────────────────────────┘          ┌──────────┐
            │                                                 │  payers  │
            │                    ENCOUNTER                    │ (Id=PK)  │
            ▼                                                 └──────────┘
     ┌──────────────┐                                               ▲
     │ observations │                                               │
     └──────────────┘                                          PAYER │
                                                                    │
                                              encounters.PAYER ─────┘

     ┌──────────────────┐
     │ US_zip_elevation │  ← joins to patients.ZIP via postalcode
     └──────────────────┘
```

### Join Keys

| From | To | Key(s) |
|---|---|---|
| `encounters` | `patients` | `PATIENT` → `patients.Id` |
| `encounters` | `payers` | `PAYER` → `payers.Id` |
| `conditions` | `patients` | `PATIENT` → `patients.Id` |
| `conditions` | `encounters` | `ENCOUNTER` → `encounters.Id` |
| `observations` | `patients` | `PATIENT` → `patients.Id` |
| `observations` | `encounters` | `ENCOUNTER` → `encounters.Id` |
| `medications` | `patients` | `PATIENT` → `patients.Id` |
| `medications` | `encounters` | `ENCOUNTER` → `encounters.Id` |
| `medications` | `payers` | `PAYER` → `payers.Id` |
| `allergies` | `patients` | `PATIENT` → `patients.Id` |
| `allergies` | `encounters` | `ENCOUNTER` → `encounters.Id` |
| `payer_transitions` | `patients` | `PATIENT` → `patients.Id` |
| `payer_transitions` | `payers` | `PAYER` → `payers.Id` |
| `US_zip_elevation` | `patients` | `postalcode` → `patients.ZIP` |
