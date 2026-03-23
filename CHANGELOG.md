# Changelog

## [6.1.0] - 2026-03-23

### Added
- Kotlin Spring Boot support (JUnit 5 + MockK) — `kotlin-springboot-guide.md`
- TypeScript/JavaScript support (Jest / Vitest) — `typescript-guide.md`
- Python support (pytest + unittest.mock) — `python-guide.md`
- Go support (testing + testify) — `go-guide.md`
- C#/.NET support (xUnit + FluentAssertions + Moq) — `csharp-guide.md`
- Auto-detection for all tech stacks (pom.xml, package.json, go.mod, *.csproj, etc.)

## [6.0.0] - 2026-03-23

### Added
- New skill: `wlb-sdet` (Software Engineer in Test)
- Converts test case documents from `wlb-test-engineer` into executable unit test code
- Java Spring Boot support (JUnit 5 + AssertJ + Mockito)
- Auto-detects project tech stack from pom.xml / build.gradle
- Maps test cases to @Test methods with Given-When-Then structure
- Supports parameterized tests for similar test cases
- Groups tests by technique (BVA, EP, Decision Table, Sequential, State Transition)
- java-springboot-guide.md with code patterns, assertion examples, and mocking patterns

## [5.1.0] - 2026-03-23

### Changed
- Workflow C: Added choice between Decision Table and Sequential Condition strategies
- Sequential Condition: fail-fast approach — if first condition fails, skip remaining conditions
- User selects condition sequence order, presented with flowchart diagram
- Sequential test table uses "Reached" column and "—" for unchecked conditions
- Fewer test cases than Decision Table (N+1 paths vs 2^N combinations)

## [5.0.0] - 2026-03-23

### Added
- Workflow D: State Transition Testing for requirements with statuses, stages, or lifecycles
- State transition diagrams (visual box-and-arrow) and state transition matrix
- Valid transition tests, invalid transition tests, and path tests (happy path, rejection path, etc.)
- Auto-detection for state/status language in requirements

## [4.0.0] - 2026-03-23

### Added
- Workflow C: Multi-Condition Analysis for requirements with multiple conditions
- Decision Table technique to combine conditions into test scenarios
- Expanded Decision Table with actual test data from BVA/EP per condition
- Auto-detection: skill analyzes the requirement and recommends the right workflow
- Conditions are decomposed, each analyzed with BVA or EP independently
- All condition analyses + Decision Table + test cases saved in one requirement file

## [3.0.0] - 2026-03-23

### Added
- Workflow B: Equivalence Partitioning (EP) for non-numeric inputs (categories, text, roles, file types)
- Visual partition diagrams showing valid/invalid sets as territories
- Workflow selection: user chooses A (BVA) or B (EP) based on input data type

### Changed
- Restructured SKILL.md as workflow router, moved detailed steps to separate files
- BVA workflow moved to `bva-workflow.md`
- EP workflow in `ep-workflow.md`
- Both workflows produce dual output tables (Unit Test + Acceptance Test)

## [2.0.0] - 2026-03-23

### Changed
- Redesigned `wlb-test-engineer` skill to focus on Boundary Value Analysis (BVA) workflow
- New 6-step workflow: identify numeric inputs → find boundaries → select possibilities → generate expected outputs → add real-world data → produce tabular test cases
- Updated reference.md to BVA-specific guidance
- Updated test-case-template.md to tabular format

## [1.0.0] - 2026-03-23

### Added
- Initial `wlb-test-engineer` skill for analyzing business requirements and generating test scenarios, test cases, and test data.
