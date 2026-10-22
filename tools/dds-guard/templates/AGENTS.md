<!-- dds-criterion: prototype-template v1 (hand-authored; in sync with satellite deployment update #2) -->
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
│       ├── SLICE.md                # the slice charter (see section 3)
│       ├── contracts.Billing/      # PUBLIC surface: DTOs, entry points. The ONLY thing neighbors may read.
│       ├── engine.Billing/         # pure decision logic. Static, no I/O, no state.
│       └── delivery.Billing/       # I/O: HTTP, DB, messaging, transactions.
└── Core/                           # near-empty: transaction handles, true shared infrastructure.
                                    # IDs and Money do NOT live here by default.
```

- Name slices by **capability / reason for change** (`Inventory`, `Billing`, `ApproveRefund`), not by entity unless the entity *is* the capability.
- Size a slice by capability and blast radius, not by table count.

## 3. Slice Charter (`SLICE.md`)

- Every slice has a `SLICE.md` inside it: 10–25 lines, **intent ONLY**.
- **HARD RULE:** the charter may only contain what code cannot express — why the slice exists, what is forbidden to it, what is NOT its problem. If a machine can derive it from the repo, it is banned from the charter.
- Allowed fields:

| Field | Purpose |
| :--- | :--- |
| `Purpose` (one line) | The capability: reasons for change live here |
| `Not here` (exclusions with owning slice) | **Anti scope-creep — the most valuable line** |
| `Language` (3–5 domain terms) | Minimal ubiquitous language |
| `Profile` + one-line why | Choice recorded at birth |
| `Surface` (rule + last-confirmed date) | "contracts.X is the only public face" |

- Prohibited: file inventories, class lists, flows, data models. **Size law:** one screen max; if it needs a TOC, it documents structure and violates the rule.
- **Re-confirmation anchor:** when `contracts.X` evolves, re-validate the charter in the same step/commit. Contract stable → charter stable.

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
- **One rule per repo, NOT per slice**: no namespace `Slices.X.engine` may depend on `Slices.Y.*` (X≠Y); no engine may depend on any delivery. New slices inherit the boundary for free.
- Mechanism: NetArchTest or ArchUnitNET as normal xunit tests, or a ~50-line custom reflection test. (If Strict is adopted: one assembly per slice + `internal` — the compiler IS the boundary.)

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
| A2 | **Contract bifurcation** (D-4): intact → contained change. Mutates → evolution event: explicit step, enumerate consumers, re-confirm charter in the same step. |
| A3 | **Modify test-first on the engine**: new plain input→output case (RED), change the pure function (GREEN). If contract mutated: adapt contracts and each consuming delivery. |
| A4 | **Boundaries**: full suite — arch tests guarantee no slice boundary was crossed and no I/O entered an engine. New path → must be claimable by the slice (orphan = defect). |
| A5 | **Close**: one work-unit commit — engine + tests + (if any) contract + re-confirmed charter, in the SAME commit. |

### Workflow B — birth a new slice

| Step | What |
| :-: | :--- |
| B0 | **Justification** (D-2): one-line Purpose without foreign internals + cluster of reasons-to-change. Anti-criteria: not by entity, not for a one-off change, if you can't write `Not here` with owners it's not a slice yet. If in doubt: ask. |
| B1 | **Charter-first, and TWO charters mutate**: the new `SLICE.md` is written BEFORE any code; the origin slice's charter gains its `Not here` line and loses what left. A birth is a re-partition of responsibility. |
| B2 | **Profile and minimal vertical scaffold**: profile at birth (Lite by default; Standard with real rules; Strict only on demonstrated pain). Scaffold = what the first use case needs; `delivery.X` is not born if there is no own I/O. |
| B3 | **Contracts before engine** (data-first): `contracts.X` designed first — the most expensive decision to revert. Cross-slice consumption resolves via contract only. |
| B4 | **Extraction test-first** (if extracted from another slice): plain input→output tests MOVE to the new namespace (RED), the pure function is extracted (GREEN). Engine purity is what makes extraction mechanical. |
| B5 | **Boundaries and birth commit**: arch tests are one generic rule per repo — the new slice inherits the boundary for free. ONE commit: new charter + updated origin charter + contracts + extracted engine + migrated tests. |

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
