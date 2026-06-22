# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `__SPECKIT_COMMAND_PLAN__` command.
See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace every placeholder with concrete project details.
  For OCR features, read 03-passport-ocr-reference-constraints.md before making
  technology or pipeline decisions.
-->

**Language/Version**: C# / .NET [record supported version, or NEEDS CLARIFICATION]

**Primary Dependencies**: [.NET libraries; Tesseract for passport OCR; React
dependencies only when UI is in scope]

**Storage**: [database/files/N/A; identify personal-data retention boundaries]

**Testing**: [.NET test framework; frontend test framework if React is used;
OCR sample-document tests when OCR behavior changes]

**Target Platform**: [server/desktop/container and supported operating systems]

**Project Type**: [backend service by default; web application only when React
frontend is in approved scope]

**Performance Goals**: [measurable, domain-specific goals]

**Constraints**: [privacy, offline/local OCR, file limits, cleanup, deployment]

**Scale/Scope**: [users, documents, screens, throughput and explicit exclusions]

**Reference Documents**: [list governing references; passport OCR features MUST
include `03-passport-ocr-reference-constraints.md`]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [ ] Backend and business logic use C#/.NET with the version and boundaries
      documented.
- [ ] React is introduced only for an approved UI and contains no backend-only
      business or OCR rules.
- [ ] Applicable reference documents are listed and their fixed decisions are
      not silently re-selected.
- [ ] Passport OCR uses local Tesseract `rus`, 300 DPI PDF rasterization,
      preprocessing, zonal OCR and manual confirm as defined by the reference.
- [ ] Partial OCR, technical failure and confirm validation are modelled as
      distinct outcomes.
- [ ] Personal-data persistence, logging and temporary-artifact cleanup comply
      with the constitution.
- [ ] Automated tests cover contracts, normalization, state transitions,
      cleanup and applicable OCR sample scenarios.
- [ ] Any violation is justified in Complexity Tracking and approved in review.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── tasks.md
```

### Source Code (repository root)

<!-- Replace the examples with one concrete structure and remove unused paths. -->

```text
src/
├── [Product].Domain/
├── [Product].Application/
├── [Product].Infrastructure/
└── [Product].Api/

tests/
├── [Product].UnitTests/
├── [Product].IntegrationTests/
└── [Product].ContractTests/

# Include only when an approved React UI exists:
frontend/
├── src/
└── tests/
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [violation] | [specific reason] | [why compliant alternative is insufficient] |
