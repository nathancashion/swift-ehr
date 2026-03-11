# Swift EHR Data Model (MSK Providers)

Local‑only EHR data model tailored to physiotherapists, chiropractors, and other MSK providers. Designed for SwiftData.

## Core Entities (SwiftData @Model)

### Patient
- **id**: UUID (MVP)
- **mrn**: String (MVP, unique)
- **firstName/lastName**: String (MVP)
- **preferredName**: String? (Ext)
- **dob**: Date (MVP)
- **sexAtBirth**: SexAtBirth (MVP)
- **genderIdentity**: GenderIdentity? (Ext)
- **phone/email**: String? (MVP)
- **address**: Address? (Ext, embedded or separate entity)
- **emergencyContactName/phone**: String? (Ext)
- **occupation**: String? (Ext)
- **dominantSide**: Laterality? (Ext)
- **notes**: String? (Ext)
- **createdAt/updatedAt**: Date (MVP)
- **relationships**: encounters [Encounter], documents [Document] (MVP)

### Provider
- **id**: UUID (MVP)
- **fullName**: String (MVP)
- **credentials**: String (MVP, e.g., PT, DPT, DC)
- **licenseNumber**: String? (Ext)
- **npi**: String? (Ext)
- **specialty**: String? (Ext)
- **email/phone**: String? (Ext)
- **relationships**: encounters [Encounter] (MVP)

### Encounter (Visit)
- **id**: UUID (MVP)
- **patient**: Patient (MVP)
- **provider**: Provider (MVP)
- **date**: Date (MVP)
- **type**: EncounterType (MVP)
- **status**: EncounterStatus (MVP)
- **chiefComplaint**: String? (MVP)
- **onsetDate**: Date? (Ext)
- **mechanismOfInjury**: String? (Ext)
- **bodyRegion**: BodyRegion? (MVP)
- **laterality**: Laterality? (MVP)
- **subjectiveNote**: String? (MVP)
- **objectiveNote**: String? (MVP)
- **assessmentNote**: String? (MVP)
- **planNote**: String? (MVP)
- **timeIn/timeOut**: Date? (Ext)
- **createdAt/updatedAt**: Date (MVP)
- **relationships**: diagnoses [Diagnosis], painAssessments [PainAssessment], romMeasurements [ROMMeasurement], treatments [Treatment], exercises [ExercisePrescription], outcomes [FunctionalOutcome], documents [Document], billingItems [BillingItem]

### Diagnosis
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **codeSystem**: CodeSystem (MVP, ICD‑10‑CM)
- **code**: String (MVP)
- **description**: String (MVP)
- **isPrimary**: Bool (MVP)
- **laterality**: Laterality? (Ext)
- **onsetDate/resolvedDate**: Date? (Ext)

### PainAssessment
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **bodyRegion**: BodyRegion (MVP)
- **laterality**: Laterality (MVP)
- **scaleType**: PainScaleType (MVP)
- **value**: Double (MVP, 0–10)
- **quality**: PainQuality? (Ext)
- **aggravatingFactors/easingFactors**: String? (Ext)
- **timing**: PainTiming? (Ext)
- **notes**: String? (Ext)

### ROMMeasurement
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **jointRegion**: BodyRegion (MVP)
- **movement**: ROMMovement (MVP, flexion/extension/etc.)
- **plane**: ROMPlane (Ext)
- **type**: ROMType (MVP, AROM/PROM)
- **valueDegrees**: Double (MVP)
- **painAtEndRange**: Bool (Ext)
- **endFeel**: EndFeel? (Ext)
- **notes**: String? (Ext)

### Treatment (In‑clinic intervention)
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **modality**: TreatmentModality (MVP)
- **bodyRegion**: BodyRegion? (MVP)
- **durationMinutes**: Int? (MVP)
- **parameters**: String? (Ext)
- **response**: String? (Ext)

### ExercisePrescription (HEP)
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **name**: String (MVP)
- **category**: ExerciseCategory (Ext)
- **sets/reps/holdSeconds**: Int? (MVP)
- **frequencyPerWeek**: Int? (MVP)
- **load/resistance**: String? (Ext)
- **cueing/instructions**: String? (MVP)

### FunctionalOutcome
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **measureType**: OutcomeMeasureType (MVP)
- **score**: Double (MVP)
- **maxScore**: Double? (Ext)
- **date**: Date (MVP)
- **interpretation**: String? (Ext)

### Document (Files/Scans/Images)
- **id**: UUID (MVP)
- **patient**: Patient (MVP)
- **encounter**: Encounter? (MVP)
- **type**: DocumentType (MVP)
- **title**: String (MVP)
- **mimeType**: String (MVP)
- **data**: Data (MVP; store with external storage)
- **createdAt**: Date (MVP)

## Additional Entities (Suggested)

### Appointment (Scheduling)
- **id**: UUID (MVP)
- **patient**: Patient (MVP)
- **provider**: Provider (MVP)
- **startAt/endAt**: Date (MVP)
- **status**: AppointmentStatus (MVP)
- **type**: AppointmentType (Ext)
- **location**: String? (Ext)
- **reason**: String? (Ext)
- **notes**: String? (Ext)
- **encounter**: Encounter? (Ext, link once completed)

### BillingItem (Charges)
- **id**: UUID (MVP)
- **encounter**: Encounter (MVP)
- **code**: String (MVP, CPT/HCPCS)
- **description**: String? (Ext)
- **units**: Int (MVP)
- **unitFee**: Decimal (Ext)
- **modifier**: String? (Ext)
- **diagnosisPointer**: [Int]? (Ext, indices to encounter diagnoses)
- **status**: BillingStatus (MVP)

### Claim (Optional, future payer workflows)
- **id**: UUID (Ext)
- **patient**: Patient (Ext)
- **encounter**: Encounter (Ext)
- **payerName**: String (Ext)
- **memberId**: String (Ext)
- **claimStatus**: ClaimStatus (Ext)
- **submittedAt/paidAt**: Date? (Ext)

### ImagingOrder (Radiology/Diagnostics)
- **id**: UUID (Ext)
- **patient**: Patient (Ext)
- **orderedBy**: Provider (Ext)
- **type**: ImagingType (Ext)
- **bodyRegion**: BodyRegion (Ext)
- **laterality**: Laterality? (Ext)
- **reason**: String? (Ext)
- **orderDate**: Date (Ext)
- **status**: ImagingStatus (Ext)
- **resultDocument**: Document? (Ext)

## Enums / Codesets (initial set)
- **SexAtBirth**: male, female, intersex, unknown
- **GenderIdentity**: man, woman, nonbinary, other, unknown
- **Laterality**: left, right, bilateral, midline
- **BodyRegion**: cervical, thoracic, lumbar, sacroiliac, shoulder, elbow, wristHand, hip, knee, ankleFoot, pelvis, rib, tmj, other
- **EncounterType**: initialEvaluation, followUp, discharge, reEval
- **EncounterStatus**: scheduled, completed, canceled, noShow
- **PainScaleType**: nrs0to10, vas, verbalDescriptor
- **PainQuality**: aching, sharp, burning, throbbing, shooting, numbness, tingling, stiffness
- **PainTiming**: constant, intermittent, activityRelated, morning, night
- **ROMType**: arom, prom, aAROM
- **ROMMovement**: flexion, extension, abduction, adduction, internalRotation, externalRotation, lateralFlexion, rotation
- **ROMPlane**: sagittal, frontal, transverse
- **EndFeel**: soft, firm, hard, empty, spasm
- **TreatmentModality**: manualTherapy, therapeuticExercise, neuroReed, dryNeedling, taping, traction, heat, ice, ultrasound, eStim, education
- **ExerciseCategory**: mobility, stability, strength, endurance, balance, proprioception
- **OutcomeMeasureType**: oswestry, ndi, lefs, dash, quickdash, faam, psfs, painCatastrophizing
- **DocumentType**: consent, intake, imaging, referral, progressNote, dischargeSummary, externalRecord
- **AppointmentStatus**: scheduled, arrived, inProgress, completed, canceled, noShow
- **AppointmentType**: eval, followUp, discharge, telehealth
- **BillingStatus**: draft, readyToSubmit, submitted, paid, denied, void
- **ClaimStatus**: created, submitted, accepted, paid, denied
- **ImagingType**: xray, mri, ct, ultrasound, emg, other
- **ImagingStatus**: ordered, scheduled, completed, resulted, canceled
- **CodeSystem**: icd10cm, snomed (future)

Codesets: **ICD‑10‑CM** for diagnoses (MVP). Optionally map to **SNOMED** later. Use **CPT/HCPCS** for billing codes (if enabled).

## Relationships (summary)
- Patient 1→* Encounters
- Provider 1→* Encounters
- Encounter 1→* Diagnoses / PainAssessments / ROMMeasurements / Treatments / ExercisePrescriptions / FunctionalOutcomes / Documents / BillingItems
- Patient 1→* Documents / Appointments / Claims / ImagingOrders

## SwiftData Notes
- Use `@Attribute(.unique)` for `Patient.mrn`.
- Use `@Attribute(.externalStorage)` for `Document.data`.
- Consider indexing: `patient.lastName`, `patient.mrn`, `encounter.date`, `diagnosis.code`, `appointment.startAt`.
- Decide on cascade delete policy early (legal retention vs local-only).

## Privacy‑Sensitive Fields (PHI)
- Patient identifiers (name, DOB, address, phone/email, MRN)
- Clinical notes and assessments
- Attachments (documents/images)
- Billing/insurance identifiers

MVP guidance: **local‑only**, **encrypt at rest**, **biometric lock**, **no network sync**.
