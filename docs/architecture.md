# Architecture

Design patterns, module boundaries, and data flow for SwiftEHR.

## Design Principles

1. **Local-Only** — All data lives on-device. No network layer, no remote APIs.
2. **MVVM + Services** — Views are declarative and stateless. ViewModels manage state. Services encapsulate business logic. SwiftData handles persistence.
3. **Protocol-Oriented** — Dependencies expressed as protocols for testability.
4. **Privacy by Default** — PHI encrypted at rest, never logged, biometric-gated.

## Layer Diagram

```
┌──────────────────────────────────────────┐
│              Views (SwiftUI)             │
├──────────────────────────────────────────┤
│         ViewModels (@Observable)         │
├──────────────────────────────────────────┤
│           Services (Business Logic)      │
├──────────────────────────────────────────┤
│        SwiftData (ModelContext)           │
├──────────────────────────────────────────┤
│     Encrypted SQLite Store (on-device)   │
└──────────────────────────────────────────┘
```

Each layer only communicates with the layer directly below it. Views never access SwiftData directly.

## Modules

### Models (`SwiftEHR/Models/`)

SwiftData `@Model` classes representing clinical entities. See [Data Models](data-model.md) for the full schema.

Core entities:
- `Patient` — Demographics, MRN, contact info
- `Provider` — Practitioner name, credentials, NPI
- `Encounter` — SOAP visit with status, chief complaint, body region
- `Diagnosis` — ICD-10-CM codes linked to encounters
- `PainAssessment` — NRS/VAS scores by body region and laterality
- `ROMMeasurement` — AROM/PROM values with end-feel
- `Treatment` — In-clinic modalities (manual therapy, e-stim, etc.)
- `ExercisePrescription` — HEP with sets/reps/hold/frequency
- `FunctionalOutcome` — Standardized outcome measures (Oswestry, NDI, etc.)
- `Document` — File attachments with external storage

Extended entities: `Appointment`, `BillingItem`, `Claim`, `ImagingOrder`

Models contain stored properties and relationships only — no business logic.

### Views (`SwiftEHR/Views/`)

SwiftUI views grouped by clinical domain:

```
Views/
├── Patient/
│   ├── PatientListView.swift
│   ├── PatientDetailView.swift
│   └── PatientFormView.swift
├── Encounter/
│   ├── EncounterListView.swift
│   ├── EncounterDetailView.swift
│   └── SOAPNoteView.swift
├── Assessment/
│   ├── PainAssessmentView.swift
│   ├── ROMMeasurementView.swift
│   └── FunctionalOutcomeView.swift
├── Treatment/
│   ├── TreatmentListView.swift
│   └── ExercisePrescriptionView.swift
├── Components/
│   ├── BodyRegionPicker.swift
│   ├── LateralityPicker.swift
│   ├── ClinicalCodeSearch.swift
│   └── PainScaleSlider.swift
└── Navigation/
    └── MainTabView.swift
```

Views read from ViewModels and dispatch actions back to them. No direct `ModelContext` access.

### ViewModels (`SwiftEHR/ViewModels/`)

`@Observable` classes that manage view state and coordinate between UI and services.

Responsibilities:
- Hold current state (loading, loaded, error)
- Transform model data into display-ready formats
- Validate user input before passing to services
- Handle search filtering and sorting

Example:

```swift
@Observable
final class PatientListViewModel {
    private let patientService: PatientServiceProtocol

    var patients: [Patient] = []
    var searchText: String = ""
    var error: EHRError?

    init(patientService: PatientServiceProtocol) {
        self.patientService = patientService
    }

    func loadPatients() throws {
        patients = try patientService.search(query: searchText)
    }
}
```

### Services (`SwiftEHR/Services/`)

Business logic layer. Each service is defined by a protocol with a concrete implementation.

Key services:
- `PatientService` — CRUD, search, demographic validation
- `EncounterService` — Visit creation, status transitions, SOAP assembly
- `DiagnosisService` — ICD-10-CM lookup, primary/secondary management
- `PainAssessmentService` — Pain score recording and trending
- `ROMMeasurementService` — ROM recording with validation
- `TreatmentService` — Modality tracking, duration logging
- `ExercisePrescriptionService` — HEP creation and management
- `OutcomeService` — Outcome measure scoring and interpretation

Services receive a `ModelContext` via dependency injection and never import SwiftUI.

### Persistence (`SwiftEHR/Persistence/`)

SwiftData configuration and schema management:

- `DataStore` — `ModelContainer` setup with encryption and file protection
- `Schema` — Version-tracked schema for migrations
- `SampleData` — Synthetic preview/test data generation

```swift
struct DataStore {
    static func container() throws -> ModelContainer {
        let schema = Schema([
            Patient.self, Provider.self, Encounter.self,
            Diagnosis.self, PainAssessment.self, ROMMeasurement.self,
            Treatment.self, ExercisePrescription.self,
            FunctionalOutcome.self, Document.self
        ])
        let config = ModelConfiguration(
            isStoredInMemoryOnly: false
        )
        return try ModelContainer(for: schema, configurations: [config])
    }
}
```

### Utilities (`SwiftEHR/Utilities/`)

- `Validators` — ICD-10-CM format, CPT code format, measurement range checks
- `Formatters` — Date display, measurement units, score formatting
- `Constants` — Reference ranges, default values, scale bounds
- `EHRError` — Unified error type with user-facing messages

## Data Flow

### Read (e.g., Load Encounters for Patient)

```
EncounterListView
  → EncounterListViewModel.loadEncounters()
    → EncounterService.getEncounters(forPatient:)
      → ModelContext.fetch(FetchDescriptor<Encounter>)
    ← [Encounter]
  ← @Observable property updated
← SwiftUI re-renders
```

### Write (e.g., Record Pain Assessment)

```
PainAssessmentView (user taps Save)
  → EncounterViewModel.savePainAssessment(...)
    → PainAssessmentService.record(...)
      → Validate: scale 0–10, body region required
      → ModelContext.insert(PainAssessment)
      → ModelContext.save()
    ← PainAssessment
  ← View dismissed, list refreshed
```

### Error Handling

```
Any View
  → ViewModel calls service method
    → Service throws EHRError
  ← ViewModel sets error property
← View displays .alert or inline error
```

## Security Architecture

### Threat Model

Since the app is local-only, the primary threats are:
- **Device theft** — mitigated by encryption at rest + biometric lock
- **Shoulder surfing** — mitigated by auto-lock on background
- **Debug leaks** — mitigated by PHI-free logging policy

### Data at Rest

- SwiftData SQLite store protected with `NSFileProtectionComplete`
- File-level encryption tied to device passcode
- `Document.data` uses `@Attribute(.externalStorage)` — also protected

### Authentication

- App launch gated by `LAContext` biometric evaluation (Face ID / Touch ID)
- Fallback to device passcode
- Configurable inactivity timeout triggers re-authentication
- No remote auth — no tokens, no OAuth, no sessions

### Logging

- Use `os.Logger` with privacy annotations
- All patient-identifiable fields marked `.private`
- No PHI in `print()`, `debugPrint()`, `dump()`, or crash reports

## Testing Strategy

- **Unit Tests** — Services and ViewModels in isolation with mock `ModelContext`
- **Integration Tests** — Full SwiftData round-trip with in-memory store
- **UI Tests** — XCUITest for critical workflows (patient creation, encounter SOAP entry)
- **Preview Data** — Synthetic `#Preview` fixtures for all views
