# AGENTS.md
> Repository: `yudao-codegen-skills`
> Last verified: 2026-03-15
> Audience: Agentic coding assistants operating in this repository

## 1) Repository scope (what this repo is)
This is a **skill-definition repository**, not a runnable application/service.
Core tracked skill artifacts at verification time (`git ls-files`):
- `SKILL.md`
- `evals/benchmark-cases.md`
- `evals/rubric.md`
- `examples/scenario-matrix.md`

Implications:
- No runnable build/lint/test configuration is present in current repository contents
- Most edits are documentation contract updates
## 2) Source-of-truth precedence
If content conflicts, follow this order:
1. `SKILL.md` (workflow, boundaries, hard-stop semantics)
2. `evals/rubric.md` (scoring thresholds and quality gates)
3. `evals/benchmark-cases.md` (P/N/C/B case expectations)
4. `examples/scenario-matrix.md` (compact scenario cues)

Do not invent behavior beyond these files unless user explicitly requests extension.
## 3) Build / lint / test commands (verified)
### 3.1 Build
- Status: **N/A in this repository**
- Reason: no runnable build configuration exists in current repository contents

### 3.2 Lint
- Status: **N/A in this repository**
- Reason: no runnable lint configuration exists in current repository contents

### 3.3 Test
- Status: **N/A in this repository**
- Reason: no runnable test configuration exists in current repository contents

### 3.4 Single test execution (important)
- Status: **N/A in this repository**
- Reason: there is no runnable test target here

### 3.5 Safe local inspection commands
- `git status`
- `git diff`
- `git ls-files`
- content lookup via grep/search tools

These commands are inspection-only and do not imply runnable build/lint/test.
## 4) About commands for real code repositories
`SKILL.md` references implementation files in external target repos (e.g., `<target-repo>/...`), including:
- `CodegenServiceImpl.java`
- `CodegenBuilder.java`
- `CodegenEngine.java`
- `CodegenTemplateTypeEnum.java`
- `CodegenFrontTypeEnum.java`
- `CodegenEngineVue3Test.java` / `CodegenEngineVue2Test.java` (mentioned in eval docs)

If a task requires build/lint/test/single-test execution:
1. Switch to the target repository first.
2. Detect actual commands from its real build files.
3. Never assume Maven/Gradle/npm commands in this skill repo.
## 5) Editing workflow in this repo
1. Read relevant sections in `SKILL.md` before editing.
2. Cross-check changed behavior against `evals/rubric.md`.
3. Update `evals/benchmark-cases.md` when trigger/boundary logic changes.
4. Keep `examples/scenario-matrix.md` aligned with scenario mapping.
5. Verify no contradictions across all four files before finishing.

When trigger rules change, ensure P/N/C/B cases remain coherent and non-conflicting.
## 6) Code style guidelines (repository-specific)
This repo is doc-centric, but style rules are strict.

### 6.1 Imports and references
- No real source-code imports are maintained in tracked files.
- In snippets, prefer fully-qualified class names/paths when ambiguous.
- Never fabricate imports, package names, or file paths.

### 6.2 Formatting
- Use ATX headers (`#`, `##`, `###`) with stable hierarchy.
- Prefer short paragraphs and bullet lists.
- Use fenced blocks with language tags when possible (`java`, `yaml`, `sql`).
- Preserve existing bilingual tone: Chinese narrative + English technical identifiers.
- Keep tables consistent (column meanings and ordering unchanged unless required).

### 6.3 Types, enums, constants
- Keep enum names/values exactly aligned with documented sources.
- Do not rename constants like `ONE`, `TREE`, `MASTER_*`, `SUB`.
- Do not introduce undocumented enum entries or inferred numeric values.

### 6.4 Naming conventions
- Preserve file naming style: `SKILL.md`, `benchmark-cases.md`, `scenario-matrix.md`.
- Keep case IDs in `P-XX`, `N-XX`, `C-XX`, `B-XX` format.
- Keep canonical Java symbol names exactly as documented.

### 6.5 Error handling semantics (content contract)
- Respect hard-stop rules from `SKILL.md`:
  - missing table comment => stop
  - missing column comments => stop
  - scenario/template mismatch => stop
  - incomplete/conflicting schema evidence => stop and ask for clarification
- Also align boundary handling with eval docs (`evals/rubric.md`, `evals/benchmark-cases.md`):
  - no primary key => continue with explicit fallback notice (first field as PK)
  - front-type conflict => stop and clarify target stack
  - no effective schema delta => avoid duplicate regeneration
  - master-sub missing sub table or invalid `subJoinColumnId` => stop
- Never downgrade hard-stop rules into soft suggestions.

### 6.6 Evidence discipline
- Every non-trivial claim must reference concrete file paths or doc anchors.
- Label inference as inference; do not present it as verified fact.
- Prefer explicit wording: “verified in this repo”.
## 7) Quality gates before completion
Before finalizing edits, verify:
- Trigger boundaries are explicit (trigger vs non-trigger)
- Workflow steps are ordered and executable
- Scenario branches (single/tree/master-sub) are clearly distinguishable
- Front-type mapping is complete and unchanged unless intentionally updated
- Boundary handling is explicit and consistent with eval docs
- No contradictions between `SKILL.md` and `evals/*`
## 8) What NOT to do
- Do not add fictional build/lint/test commands for this repo.
- Do not claim a local single-test command exists here.
- Do not silently broaden scope beyond Yudao/RuoYi-Vue-Pro codegen context.
- Do not mutate “template-driven codegen skill” into generic CRUD guidance.
## 9) Cursor/Copilot rules status
Checked:
- `.cursorrules`
- `.cursor/rules/`
- `.github/copilot-instructions.md`

Result: **none of these files exist in this repository**.
Therefore, this `AGENTS.md` is the primary **repository-local** instruction file,
subject to higher-level system/global instructions.
## 10) Minimal handoff template
When handing off, include:
1. What changed (file paths + intent)
2. Which source-of-truth sections were used
3. Any unresolved ambiguity
4. Whether P/N/C/B cases are impacted
5. Whether downstream target-repo verification is required

Keep handoff concrete, path-based, and falsifiable.
