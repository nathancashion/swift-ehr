# Contributing to SwiftEHR

Guidelines for contributing code, documentation, and bug reports.

## Code of Conduct

Be respectful, constructive, and professional in all interactions.

## Getting Started

See the [Setup Guide](docs/setup-guide.md) for environment setup.

### Prerequisites

- Xcode 15.0+
- Swift 5.9+
- iOS 17.0+ Simulator or device
- SwiftLint (`brew install swiftlint`)

## Code Style

### Swift Conventions

- Follow the [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/).
- Use `SwiftLint` for automated enforcement (`.swiftlint.yml` in repo root).
- Prefer value types (`struct`, `enum`) for data; use `class` only for SwiftData `@Model` types and `@Observable` view models.
- Use `async/await` for asynchronous code.
- Mark classes as `final` unless designed for subclassing.

### Naming

- Types and protocols: `UpperCamelCase` — `PatientService`, `EncounterStatus`
- Functions, variables, properties: `lowerCamelCase` — `fetchPatient(id:)`, `bodyRegion`
- Enum cases: `lowerCamelCase` — `initialEvaluation`, `arom`, `nrs0to10`
- Abbreviations as words: `romMeasurement`, `npiNumber`, `icd10Code`

### File Organization

- One primary type per file, named to match: `Patient.swift`, `EncounterService.swift`.
- Extensions in `TypeName+Category.swift`: `Patient+Validation.swift`.
- Use `// MARK: -` to organize sections within a file.
- Group files by domain in the directory structure (Models/, Views/Patient/, etc.).

### Documentation

Add `///` doc comments to all public types, methods, and properties:

```swift
/// Fetches all encounters for a patient, ordered by date descending.
///
/// - Parameter patientId: The UUID of the patient.
/// - Returns: An array of encounters sorted by most recent first.
/// - Throws: `EHRError.notFound` if the patient does not exist.
func getEncounters(forPatient patientId: UUID) throws -> [Encounter]
```

## Commit Conventions

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short summary>
```

### Types

- `feat` — New feature
- `fix` — Bug fix
- `docs` — Documentation only
- `style` — Formatting, no logic change
- `refactor` — Code restructuring, no feature/fix
- `test` — Adding or updating tests
- `chore` — Build, CI, tooling

### Examples

```
feat(encounter): add SOAP note text fields
fix(rom): correct flexion range validation
docs(readme): add quick start instructions
test(patient): add unit tests for MRN uniqueness
```

## Pull Request Process

1. Create a branch from `master`: `git checkout -b feature/your-feature`
2. Make changes following the style guidelines.
3. Add/update tests for new functionality.
4. Run tests: **⌘U** in Xcode.
5. Run `swiftlint` and fix any warnings.
6. Commit with conventional commit messages.
7. Push and open a PR against `master`.

### PR Checklist

- [ ] Code follows project style guidelines
- [ ] Self-reviewed the diff
- [ ] Added `///` doc comments for public APIs
- [ ] Added or updated tests
- [ ] All tests pass locally
- [ ] No new SwiftLint warnings
- [ ] Updated documentation if applicable

## Healthcare-Specific Rules

SwiftEHR handles Protected Health Information (PHI). These rules are **non-negotiable**:

### Never Log PHI

- **Never** print, log, or `dump()` patient data — not even in debug builds.
- Use `os.Logger` with `.private` redaction for any identifiable fields.
- Strip PHI from crash reports and analytics.

### Synthetic Test Data Only

- **Never** use real patient data in unit tests, UI tests, previews, or screenshots.
- Use clearly synthetic data: "Jane Doe", DOB 2000-01-01, MRN "TEST-0001".
- Mark test fixtures with comments indicating they are synthetic.

### Secure Storage

- Store any credentials or sensitive config in the iOS **Keychain**.
- **Never** store PHI in `UserDefaults`, plain files, or unprotected storage.
- Use `NSFileProtectionComplete` for the SwiftData store.

### Biometric Authentication

- Gate app access behind Face ID / Touch ID via `LAContext`.
- Re-authenticate after configurable inactivity timeout.
- Fall back to device passcode if biometrics unavailable.

### Input Validation

- Validate clinical codes (ICD-10-CM, CPT/HCPCS) against expected formats.
- Validate measurement ranges (pain 0–10, ROM 0–360°).
- Sanitize free-text inputs to prevent injection in any future export paths.

## Reporting Bugs

Open a [GitHub issue](https://github.com/nathancashion/swift-ehr/issues) with:
- Steps to reproduce
- Expected vs. actual behavior
- iOS version, device model, Xcode version
- Screenshots if applicable

Or use `bd create` if you have beads set up (see `AGENTS.md`).
