# SwiftEHR

A local-only Electronic Health Record (EHR) app for musculoskeletal (MSK) providers, built with Swift, SwiftUI, and SwiftData.

## Overview

SwiftEHR gives physiotherapists, chiropractors, and other MSK providers a fast, privacy-first charting tool on iOS and iPadOS. All data stays on-device — no cloud sync, no server dependency. Designed around the SOAP note workflow with structured clinical assessments built in.

### Features

**MVP**
- Patient demographics and medical record management
- SOAP encounter documentation (Subjective, Objective, Assessment, Plan)
- ICD-10-CM diagnosis coding with primary/secondary designation
- Pain assessments (NRS 0–10, VAS, verbal descriptor) by body region
- Range of motion measurements (AROM/PROM) with end-feel documentation
- In-clinic treatment tracking (manual therapy, therapeutic exercise, modalities)
- Home exercise prescription (HEP) with sets/reps/hold/frequency
- Functional outcome measures (Oswestry, NDI, LEFS, DASH, PSFS, etc.)
- Document attachments (consent forms, imaging, referrals, progress notes)

**Extended**
- Appointment scheduling with status tracking
- Billing items (CPT/HCPCS) with diagnosis pointers
- Insurance claims workflow
- Imaging/diagnostic orders
- Gender identity and preferred name support
- Mechanism of injury and onset documentation

### Security & Privacy

- **Local-only** — no network sync, no cloud storage
- **Encrypted at rest** — SwiftData store with file protection
- **Biometric lock** — Face ID / Touch ID gating
- **No PHI logging** — patient data never written to logs or console

## Requirements

| Component    | Minimum Version |
|--------------|-----------------|
| iOS / iPadOS | 17.0            |
| Xcode        | 15.0            |
| Swift        | 5.9             |
| macOS (dev)  | Ventura 13.5    |

## Quick Start

```bash
git clone https://github.com/nathancashion/swift-ehr.git
cd swift-ehr
open SwiftEHR.xcodeproj
```

Select an iPhone or iPad simulator (or device), then press **⌘R** to build and run.

See the [Setup Guide](docs/setup-guide.md) for detailed instructions.

## Project Structure

```
swift-ehr/
├── SwiftEHR/
│   ├── Models/          # SwiftData @Model entities (Patient, Encounter, etc.)
│   ├── Views/           # SwiftUI views organized by clinical domain
│   ├── ViewModels/      # @Observable view models (MVVM)
│   ├── Services/        # Business logic and domain services
│   ├── Persistence/     # SwiftData configuration and migrations
│   └── Utilities/       # Extensions, validators, formatters
├── SwiftEHRTests/       # Unit and integration tests
├── SwiftEHRUITests/     # UI tests
├── docs/                # Project documentation
└── .beads/              # Issue tracking (bd)
```

## Architecture

MVVM with a SwiftData persistence layer:

```
View (SwiftUI) → ViewModel (@Observable) → Service → SwiftData ModelContext
```

See [Architecture](docs/architecture.md) for details.

## Documentation

- [Setup Guide](docs/setup-guide.md) — Installation, configuration, first run
- [Architecture](docs/architecture.md) — Design patterns, modules, data flow, security
- [Data Models](docs/data-model.md) — SwiftData entities, enums, relationships
- [API Reference](docs/api-reference.md) — Service protocol interfaces
- [Contributing](CONTRIBUTING.md) — Code style, commits, PR process

## Issue Tracking

This project uses [bd (beads)](https://github.com/beads-dev/beads) for issue tracking. See `AGENTS.md` for workflow details.

```bash
bd ready          # Find available work
bd show <id>      # View issue details
bd close <id>     # Complete work
```

## Roadmap

**Phase 1 — Core Charting (MVP)**
- [ ] Patient CRUD and search
- [ ] Encounter creation with SOAP notes
- [ ] Diagnosis, pain, ROM, treatment, exercise, outcome entry
- [ ] Document attachment and viewing

**Phase 2 — Practice Management**
- [ ] Appointment scheduling and calendar view
- [ ] Billing item entry with CPT codes
- [ ] Provider management

**Phase 3 — Reporting & Polish**
- [ ] Outcome trending charts
- [ ] Encounter history timeline
- [ ] PDF report generation
- [ ] Apple Pencil body diagram annotations

**Phase 4 — Future**
- [ ] Insurance claims workflow
- [ ] Imaging order tracking
- [ ] Data export (CSV, FHIR bundle)
- [ ] watchOS vitals companion

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
