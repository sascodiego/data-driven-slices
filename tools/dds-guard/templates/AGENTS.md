<!-- dds-criterion: prototype-template v2 (hand-authored; includes definition trio + seal protocol from cluster C8+C9) -->
<!-- Placeholders: {{PROJECT_NAME}}, {{PROJECT_DESCRIPTION}}, {{SOLUTION_LAYOUT}} — fill per project. -->

# AGENTS.md — {{PROJECT_NAME}}

{{PROJECT_DESCRIPTION}}

Stack: C# / .NET 8+. This repository uses **Data-Driven Slices (DDS)** as its architectural criterion. The instructions below are **self-contained**: follow them without assuming external DDS documentation.

---

## 1. The DDS Criterion (one paragraph)

Separate **Data** (what is known) from **Decision** (what is computed) from **Delivery** (what changes the world). A **Slice** is a vertical, self-contained unit of capability. Business logic lives in pure, static **Engines** that receive and return plain data. Slices are the unit of change, of exploration, and of blast-radius control.

## 2. Slice Layout & Naming

```text
/src
├── Slices/
│   └── Billing/                    # one slice = one capability (NOT one table)
│       ├── SLICE.md                # the slice charter: clauses as data (see section 3)
│       ├── Billing.slice.test.cs   # definition test: derived rules + embedded clause copy
│       ├── slice.shax              # seal: sha256 of the SLICE.md + definition-test pair
│       ├── contracts.Billing/      # PUBLIC surface: DTOs, entry points. The ONLY thing neighbors may read.
│       ├── engine.Billing/         # pure decision logic. Static, no I/O, no state.
│       └── delivery.Billing/       # I/O: HTTP, DB, messaging, transactions.
└── Core/                           # near-empty: transaction handles, true shared infrastructure.
                                    # IDs and Money do NOT live here by default.
```

- Name slices by **capability / reason for change** (`Inventory`, `Billing`, `ApproveRefund`), not by entity unless the entity *is* the capability.
- Size a slice by capability and blast radius, not by table count.
- All `*.slice.test.cs` files compile into one repo-level definition-tests project (glob include).

## 3. Slice Definition Trio (charter + definition test + seal)

Every slice carries three definition files at its root. **A slice without the trio does not exist.**

### `SLICE.md` — the charter as data

```markdown
---
name: Billing
purpose: own everything about charging money for completed sales
excludes:
  - { concept: pricing, owner: Pricing, decision: DDS-012 }
language:
  - { term: Charge, means: attempt to charge money }
consumes: [Pricing]
profile: { level: Standard, decision: DDS-011 }
surface: contracts.Billing
confirmed: 2026-10-22
---
```

- **HARD RULE:** the charter may only contain what code cannot express — why the slice exists, what is forbidden to it, what is NOT its problem, whom it consumes. If a machine can derive it from the repo, it is banned.
- Prohibited: file inventories, class lists, flows, data models. **Size law:** one screen max.
- The WHY lives in the project decision log (anchored table), referenced by decision IDs — never inside the charter.

### `[Slice].slice.test.cs` — the enforcement

- Structural rules **derived from the clauses**, carrying an **embedded copy** of the declaration they enforce (never trusts the `.md` at runtime).
- Consistency check: embedded copy ↔ front-matter. A charter edited without re-deriving the test goes red.

### `slice.shax` — the pair seal

```
sha256( sha256(SLICE.md) + sha256([Slice].slice.test.cs) )
```

One hex line. **One seal for the pair:** any modification to either file breaks the seal → instant red on `dotnet test` → the change must be explained. Rotation is legitimate ONLY via the definition protocol (below) plus a decision-log row.

### Definition protocol (impact gate)

| Level | What changes | Touches | Does NOT touch |
| :-: | :--- | :--- | :--- |
| **3. Definition** | What the slice IS/FOR: purpose, excludes, consumes, language, profile | Charter + definition test + `.shax` + log row | — |
| **2. Promise** | The surface (`contracts.X`) without a concept change | Contract evolution event (D-4) + `confirmed` re-stamp | The trio |
| **1. Implementation** | Engine logic, delivery internals | Engine + behavior tests | **Neither the `.md`, nor its test, nor the seal** |

Level 1 is the common case: the trio is untouched (zero friction). Level 3 rotation: edit clauses → re-derive test → recompute the pair seal (`sha256sum` on both files) → update `.shax` → append decision-log row. Detection is state-based: half-done rotations stay red until completed.

### Bug triage (when the task is a bug)

The report is the spec of the desire → write the RED test first. Then classify against the truth stack (charter → contract → test → code): is the defect in the implementation (code ≠ its test), the promise (contract lies), or the definition (charter lies)? Fix at the classified layer; the change cascades down. **Hard rule: never edit an assertion to make the suite green without justifying in the same commit why the old expected behavior was wrong.**

## 4. Profiles (structure is proportional to risk, not dogma)

| Profile | Structure | Use when | Enforcement |
| :--- | :--- | :--- | :--- |
| **Lite** | Conceptual `Data → Decision → Delivery`, minimal files, no separate assemblies | Simple CRUD, prototypes, low-risk domains | Convention only |
| **Standard** | Explicit `contracts / engine / delivery` files | Real business rules, growing test suites | Convention + clear files |
| **Strict** | One assembly per slice, `internal` visibility | High-risk, multi-team/multi-agent, recurring boundary violations | **Compiler-enforced** |

- Record the chosen profile **when the slice is created** (in the charter). Changing it later is a promotion/demotion decision, not a refactor whim.
- **Promote only on concrete pain**: cross-slice violations recurring, tests mixing I/O with rules, blast radius growing uncontrolled. No pain → no promotion.

## 5. Boundary Rules (hard)

1. **Read/write within your slice only.** A task targeting slice `X` modifies files under `Slices/X/` (plus its own tests). Nothing else.
2. **Cross-slice consumption = contracts only.** To coordinate with slice `Y`, read `contracts.Y`. **Never** read or reference `engine.Y` / `delivery.Y` internals.
3. **`contracts.*` changes are contract evolution events.** Changing a public contract shape requires an explicit, separately-visible step (not smuggled inside an unrelated task).
4. **`Core/` is infrastructure only.** Touching it requires justification in the task. IDs and Money do not go in Core by default.
5. **New files must be claimable.** Every new path must belong to a slice (or be explicitly mapped). Orphan files are a defect.

## 6. Testing (Zero-Mocks for Engines)

- **Engine tests**: pure input → output assertions on flat data. **No dynamic mocks** (no Moq/NSubstitute for engines), no DB, no network. These suites must run in milliseconds.
- **Delivery tests**: may use controlled test doubles, fakes, testcontainers, or fake HTTP servers **when validating observable behavior** (not internal call choreography).
- A failing engine test is located by comparing flat input/output data — never by debugging interaction setups.

### Architecture Tests (executable boundaries)

- Boundary rules must run with the normal test suite and fail red like any test.
- **One rule per repo, NOT per slice**: no namespace `Slices.X.engine` may depend on `Slices.Y.*` (X≠Y); no engine may depend on any delivery; **cross-slice references must be declared in `consumes`** — declared set must equal the actual set. New slices inherit the boundary for free.
- Mechanism: NetArchTest or ArchUnitNET as normal xunit tests, or a ~50-line custom reflection test. (If Strict is adopted: one assembly per slice + `internal` — the compiler IS the boundary.)
- **Seal test (generic, repo-level):** for every slice, hash `SLICE.md` and its definition test, recompute the pair seal, compare against `slice.shax`, and require the full trio to exist.

## 7. Decision Matrix (how a change enters the system)

Questions are consulted **in order**; each answer derives to a workflow or the next question.

| # | Question | Answer → action |
| :-: | :--- | :--- |
| D-1 | Does the change fit the `Purpose` of an existing slice? (answer by reading **charters**, not code) | Yes → Workflow A. No/partial → D-2. |
| D-2 | Does the new capability have its own cluster of reasons-to-change, and a one-line `Purpose` that doesn't mention other slices' internals? | Yes → Workflow B. No → nearest existing slice (Workflow A) and **record the pain**: candidate for profile promotion or future slice. |
| D-3 | Does the change cross several slices? | Yes → multi-slice: decompose per slice, contract as the coordination boundary. No → continue. |
| D-4 | Does the public surface (`contracts.X`) mutate? | Yes → **CONTRACT EVOLUTION EVENT**: explicit, separately-visible step + re-confirm charter(s) + enumerate consumers (namespace usages). No → change contained in the slice. |
| D-5 | New I/O (HTTP, DB, messaging)? | Yes → touches `delivery.X`, with observable-behavior tests. No → engine only. |
| D-6 | Is the slice profile still proportional to its risk? | Re-evaluate **only on observable pain** (promotion by symptoms). No pain → no change. |
| D-7 | No charter resolves the routing doubt? | **ASK THE HUMAN.** 30 seconds of question before a change in the wrong slice. |

## 8. Operational Workflows

### Workflow A — modify behavior of an existing feature

| Step | What |
| :-: | :--- |
| A0 | **Routing charter-first**: read `SLICE.md` of candidate slices; `Not here` decides. Unresolvable doubt → D-7 (ask). |
| A1 | **See the current feature**, in order: `contracts.X` (surface) → engine tests (executable spec of current behavior) → `engine.X` (only now the code). |
| A2 | **Impact gate**: does the change reach the definition, the contract, or stay contained? Contained → A3 (trio untouched). Contract → evolution event + `confirmed` re-stamp. Definition → full definition protocol (charter + test + seal + log row) — and if the definition changes because "this belongs elsewhere": classify (drift 9a/9b/9c) and ask. |
| A2b | **Bug triage** (if the task is a bug): the report is the spec → RED first; classify against the truth stack (implementation / promise / definition); fix at the classified layer. Never edit an assertion to green without same-commit justification. |
| A3 | **Modify test-first on the engine**: new plain input→output case (RED), change the pure function (GREEN). If contract mutated: adapt contracts and each consuming delivery. |
| A4 | **Boundaries**: full suite — arch tests guarantee no slice boundary was crossed and no I/O entered an engine. New path → must be claimable by the slice (orphan = defect). |
| A5 | **Close**: one work-unit commit — engine + tests + (if any) contract + re-confirmed charter, in the SAME commit. |

### Workflow B — birth a new slice

| Step | What |
| :-: | :--- |
| B0 | **Justification** (D-2): one-line Purpose without foreign internals + cluster of reasons-to-change. Anti-criteria: not by entity, not for a one-off change, if you can't write `Not here` with owners it's not a slice yet. If in doubt: ask. |
| B1 | **Trio-first, and TWO charters mutate**: the new trio (`SLICE.md` + definition test + `slice.shax`) is written BEFORE any business code; the origin slice's charter gains its `Not here` line and loses what left (its definition test re-derived and re-sealed). A birth is a re-partition of responsibility. |
| B2 | **Profile and minimal vertical scaffold**: profile at birth (Lite by default; Standard with real rules; Strict only on demonstrated pain). Scaffold = what the first use case needs; `delivery.X` is not born if there is no own I/O. |
| B3 | **Contracts before engine** (data-first): `contracts.X` designed first — the most expensive decision to revert. Cross-slice consumption resolves via contract only. |
| B4 | **Extraction test-first** (if extracted from another slice): plain input→output tests MOVE to the new namespace (RED), the pure function is extracted (GREEN). Engine purity is what makes extraction mechanical. |
| B5 | **Boundaries and birth commit**: arch tests are one generic rule per repo — the new slice inherits the boundary for free. ONE commit: new sealed trio + re-sealed origin trio + contracts + extracted engine + migrated tests. The slice is born sealed. |

### Multi-slice changes (D-3 = yes)

1. Decompose per slice (matrix from D-1 for each).
2. The contract is the coordination boundary: never share engine or delivery.
3. Commit per slice, or one coordinated commit for a shared contract step — never a commit mixing two slices' internals.
4. If coordination reveals a stable shared shape: evaluate D-2 (own slice vs consumer contract). Do NOT create a slice just to "share data".

## 9. Conventions

- Commits: Conventional Commits (`feat(billing): ...`, `fix(inventory): ...`). One work unit per task.
- Branch from `main` for feature work; never commit directly to `main`.
- Tests and docs travel **with** the behavior change, in the same work unit.

## 10. Build & Test ({{PROJECT_NAME}})

```bash
dotnet build                        # whole solution
dotnet test                         # all tests (engines must be milliseconds-fast)
dotnet test --filter "FullyQualifiedName~engine"   # engine-only fast suite
```

## 11. Do NOT

- Do not create interfaces/abstractions "for future flexibility" inside a slice — Data/Decision/Delivery separation is the structure.
- Do not hide state behind objects; transport plain data through engines.
- Do not mix I/O into engine logic — that code belongs in delivery.
- Do not read a neighbor slice's engine or delivery code, even "just to look".
- Do not promote a slice's profile without observed pain.
- Do not put structure (file lists, flows, models) into a charter — intent only.
- Do not edit a test assertion to make the suite green without classifying it as a spec change in the same commit.
