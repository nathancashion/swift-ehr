# API Reference

Service protocol interfaces for SwiftEHR. All services operate against the local SwiftData store via `ModelContext`.

## Error Types

```swift
enum EHRError: Error, LocalizedError {
    case notFound(entity: String, id: UUID)
    case validationFailed(field: String, message: String)
    case duplicateMRN(String)
    case invalidCode(system: String, code: String)
    case measurementOutOfRange(field: String, value: Double, range: ClosedRange<Double>)
    case persistenceFailed(underlying: Error)

    var errorDescription: String? { ... }
}
```

---

## PatientService

```swift
protocol PatientServiceProtocol {
    /// Search patients by name or MRN.
    func search(query: String) throws -> [Patient]

    /// Fetch a patient by ID.
    func get(id: UUID) throws -> Patient

    /// Create a new patient. Validates MRN uniqueness.
    /// - Throws: `EHRError.duplicateMRN` if MRN already exists.
    func create(_ patient: Patient) throws -> Patient

    /// Update an existing patient record.
    func update(_ patient: Patient) throws -> Patient

    /// Delete a patient and all associated records.
    func delete(id: UUID) throws

    /// Fetch all encounters for a patient, ordered by date descending.
    func encounters(forPatient id: UUID) throws -> [Encounter]

    /// Fetch all documents for a patient.
    func documents(forPatient id: UUID) throws -> [Document]
}
```

---

## EncounterService

```swift
protocol EncounterServiceProtocol {
    /// Create a new encounter for a patient with a provider.
    func create(
        patient: Patient,
        provider: Provider,
        type: EncounterType,
        date: Date
    ) throws -> Encounter

    /// Fetch an encounter by ID.
    func get(id: UUID) throws -> Encounter

    /// Update encounter fields (SOAP notes, chief complaint, body region, etc.).
    func update(_ encounter: Encounter) throws -> Encounter

    /// Transition encounter status.
    /// Valid transitions: scheduled → completed, scheduled → canceled, scheduled → noShow.
    /// - Throws: `EHRError.validationFailed` for invalid transitions.
    func updateStatus(_ id: UUID, to status: EncounterStatus) throws -> Encounter

    /// Delete an encounter and all child records.
    func delete(id: UUID) throws

    /// Fetch all assessments, treatments, and prescriptions for an encounter.
    func summary(forEncounter id: UUID) throws -> EncounterSummary
}

struct EncounterSummary {
    let encounter: Encounter
    let diagnoses: [Diagnosis]
    let painAssessments: [PainAssessment]
    let romMeasurements: [ROMMeasurement]
    let treatments: [Treatment]
    let exercises: [ExercisePrescription]
    let outcomes: [FunctionalOutcome]
}
```

---

## DiagnosisService

```swift
protocol DiagnosisServiceProtocol {
    /// Add a diagnosis to an encounter.
    /// - Parameter isPrimary: If true, demotes any existing primary diagnosis.
    func add(
        code: String,
        description: String,
        codeSystem: CodeSystem,
        isPrimary: Bool,
        laterality: Laterality?,
        toEncounter encounterId: UUID
    ) throws -> Diagnosis

    /// Remove a diagnosis from an encounter.
    func remove(id: UUID) throws

    /// Update diagnosis fields (primary status, laterality, dates).
    func update(_ diagnosis: Diagnosis) throws -> Diagnosis

    /// Search ICD-10-CM codes by keyword or code prefix.
    func searchCodes(query: String) throws -> [(code: String, description: String)]
}
```

---

## PainAssessmentService

```swift
protocol PainAssessmentServiceProtocol {
    /// Record a pain assessment for an encounter.
    /// - Throws: `EHRError.measurementOutOfRange` if value is outside 0–10.
    func record(
        encounter: Encounter,
        bodyRegion: BodyRegion,
        laterality: Laterality,
        scaleType: PainScaleType,
        value: Double,
        quality: PainQuality?,
        aggravatingFactors: String?,
        easingFactors: String?,
        timing: PainTiming?,
        notes: String?
    ) throws -> PainAssessment

    /// Fetch pain assessment history for a patient and body region.
    /// Useful for trending pain over time.
    func history(
        forPatient patientId: UUID,
        bodyRegion: BodyRegion?,
        laterality: Laterality?
    ) throws -> [PainAssessment]

    /// Delete a pain assessment.
    func delete(id: UUID) throws
}
```

---

## ROMMeasurementService

```swift
protocol ROMMeasurementServiceProtocol {
    /// Record a range of motion measurement.
    /// - Throws: `EHRError.measurementOutOfRange` if degrees outside 0–360.
    func record(
        encounter: Encounter,
        jointRegion: BodyRegion,
        movement: ROMMovement,
        type: ROMType,
        valueDegrees: Double,
        plane: ROMPlane?,
        painAtEndRange: Bool?,
        endFeel: EndFeel?,
        notes: String?
    ) throws -> ROMMeasurement

    /// Fetch ROM history for a patient, joint, and movement.
    func history(
        forPatient patientId: UUID,
        jointRegion: BodyRegion?,
        movement: ROMMovement?
    ) throws -> [ROMMeasurement]

    /// Delete a ROM measurement.
    func delete(id: UUID) throws
}
```

---

## TreatmentService

```swift
protocol TreatmentServiceProtocol {
    /// Record an in-clinic treatment for an encounter.
    func record(
        encounter: Encounter,
        modality: TreatmentModality,
        bodyRegion: BodyRegion?,
        durationMinutes: Int?,
        parameters: String?,
        response: String?
    ) throws -> Treatment

    /// Update treatment details.
    func update(_ treatment: Treatment) throws -> Treatment

    /// Delete a treatment record.
    func delete(id: UUID) throws
}
```

---

## ExercisePrescriptionService

```swift
protocol ExercisePrescriptionServiceProtocol {
    /// Prescribe a home exercise for an encounter.
    func prescribe(
        encounter: Encounter,
        name: String,
        category: ExerciseCategory?,
        sets: Int?,
        reps: Int?,
        holdSeconds: Int?,
        frequencyPerWeek: Int?,
        load: String?,
        instructions: String?
    ) throws -> ExercisePrescription

    /// Update an exercise prescription.
    func update(_ exercise: ExercisePrescription) throws -> ExercisePrescription

    /// Fetch the current HEP (all active exercises) for a patient.
    func currentHEP(forPatient patientId: UUID) throws -> [ExercisePrescription]

    /// Delete an exercise prescription.
    func delete(id: UUID) throws
}
```

---

## OutcomeService

```swift
protocol OutcomeServiceProtocol {
    /// Record a functional outcome measure score.
    func record(
        encounter: Encounter,
        measureType: OutcomeMeasureType,
        score: Double,
        maxScore: Double?,
        date: Date,
        interpretation: String?
    ) throws -> FunctionalOutcome

    /// Fetch outcome history for a patient and measure type.
    /// Useful for tracking progress over time.
    func history(
        forPatient patientId: UUID,
        measureType: OutcomeMeasureType?
    ) throws -> [FunctionalOutcome]

    /// Calculate the minimal clinically important difference (MCID)
    /// between two scores for a given measure type.
    func evaluateChange(
        from baseline: FunctionalOutcome,
        to current: FunctionalOutcome
    ) -> OutcomeChangeEvaluation

    /// Delete an outcome record.
    func delete(id: UUID) throws
}

enum OutcomeChangeEvaluation {
    case clinicallySignificantImprovement
    case minimalImprovement
    case noChange
    case decline
}
```

---

## Supporting Enums

These match the enums defined in [Data Models](data-model.md):

- `EncounterType` — `initialEvaluation`, `followUp`, `discharge`, `reEval`
- `EncounterStatus` — `scheduled`, `completed`, `canceled`, `noShow`
- `BodyRegion` — `cervical`, `thoracic`, `lumbar`, `sacroiliac`, `shoulder`, `elbow`, `wristHand`, `hip`, `knee`, `ankleFoot`, `pelvis`, `rib`, `tmj`, `other`
- `Laterality` — `left`, `right`, `bilateral`, `midline`
- `PainScaleType` — `nrs0to10`, `vas`, `verbalDescriptor`
- `PainQuality` — `aching`, `sharp`, `burning`, `throbbing`, `shooting`, `numbness`, `tingling`, `stiffness`
- `PainTiming` — `constant`, `intermittent`, `activityRelated`, `morning`, `night`
- `ROMType` — `arom`, `prom`, `aAROM`
- `ROMMovement` — `flexion`, `extension`, `abduction`, `adduction`, `internalRotation`, `externalRotation`, `lateralFlexion`, `rotation`
- `EndFeel` — `soft`, `firm`, `hard`, `empty`, `spasm`
- `TreatmentModality` — `manualTherapy`, `therapeuticExercise`, `neuroReed`, `dryNeedling`, `taping`, `traction`, `heat`, `ice`, `ultrasound`, `eStim`, `education`
- `ExerciseCategory` — `mobility`, `stability`, `strength`, `endurance`, `balance`, `proprioception`
- `OutcomeMeasureType` — `oswestry`, `ndi`, `lefs`, `dash`, `quickdash`, `faam`, `psfs`, `painCatastrophizing`
- `CodeSystem` — `icd10cm`, `snomed`
