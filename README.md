# Data-Driven Slices (DDS) & Data-Oriented Design (DoD)
## A Proposed Architectural Criterion for Agentic AI Coding (Prospectus / Manifesto)

Repository name: `data-driven-slices`

> [!NOTE]
> **Status: prospectus, not a proven framework.** This repository proposes **Data-Driven Slices (DDS)** as an architectural *criterion* and a set of **measurable hypotheses** for agentic AI coding. The criterion is intended to be broadly applicable, but the benefits are **not yet validated** — they are to be tested through worked examples and benchmarks (see [DDS as a Measurable Prospectus](#dds-as-a-measurable-prospectus-not-a-proven-framework)).

This repository documents the **Data-Driven Slices (DDS)** architectural pattern combined with a **Data-Oriented Design (DoD)** posture. As a **working hypothesis**, DDS is proposed as an alternative to dependency-heavy clean architectures and classic Object-Oriented Programming (OOP), with the goal of improving both execution characteristics and the autonomous development experience of **AI Coding Agents**.

> [!NOTE]
> **Language Status:** The core architectural documentation is currently written and actively expanding in **Spanish** (`docs/es/`) to refine terms and gather feedback. It will be translated and fully formalized in English in its final release.

---

## DDS Is a Criterion, Not a Ceremony

DDS is proposed as a **portable decision criterion**, not because every project must adopt the same structure. The same idea — *separate **Data** (what is known) from **Decision** (what is computed) from **Delivery** (what changes the world)* — is *expected to be worth testing* in both a 50-line CRUD service and a multi-team billing platform. What changes is **how much physical structure** that separation requires, and that is driven by **risk, not dogma**. Broad applicability is a hypothesis to validate (see [the prospectus](#dds-as-a-measurable-prospectus-not-a-proven-framework)), not an established fact.

The core intent is preserved at every level: **bounded scope, TDD-friendliness, lower AI exploration cost, and refactor blast-radius control** — these are the outcomes DDS *aims to* produce, and the outcomes that must be measured (see the prospectus below).

### Progressive Profiles

| Profile | Structure | When to use it | Boundary enforcement |
| :--- | :--- | :--- | :--- |
| **DDS Lite** | Conceptual `Data → Decision → Delivery` separation. Minimal files, **no** separate assemblies. | Simple CRUD, small services, prototypes, low-risk domains. | Convention only. Pure `Decision` extraction is applied **only where it reduces risk**. |
| **DDS Standard** | Explicit `contracts / engine / delivery` files with clear data flow. | Business logic with real rules, growing test suites, or AI exploration that benefits from bounded context. | Convention + clear files. Separate assemblies are **recommended, not mandatory**. |
| **DDS Strict** | Physical boundaries: one assembly per slice, `internal` visibility, compiler-enforced isolation. | High-risk / extreme projects, multi-team or multi-agent work where convention is insufficient. | **Compiler-enforced.** |

> The detailed reference in [`DDS_Metodo_Trabajo_IA.md`](docs/es/DDS_Metodo_Trabajo_IA.md) documents the **Strict** profile by default (the strongest form). Lite and Standard are legitimate, first-class choices — not "incomplete" Strict.

### Promotion Triggers (symptom-driven, not schedule-driven)

Promotion is **only** justified by concrete pain or risk. If there is no pain, **do not promote.**

| From → To | Promote when you observe… |
| :--- | :--- |
| **Lite → Standard** | Decision / data / effects no longer fit clearly in a simple unit · tests mix I/O with rules · the AI must read too much to act · delivery starts contaminating decision logic. |
| **Standard → Strict** | Cross-slice imports or boundary violations recur · refactor blast radius grows uncontrollably · multi-agent or multi-team coordination requires compiler enforcement. |

---

## DDS as a Measurable Prospectus (Not a Proven Framework)

DDS is published as a **prospectus / manifesto of design criteria and measurable hypotheses**, not as a validated, universal framework. The *criterion* is intended to be broadly applicable; the *evidence* for its benefits is pending. This section states what DDS proposes, what it does **not** claim yet, and what must be measured before any hypothesis is promoted to a result.

### What DDS proposes
- A **decision criterion**: separate **Data** (what is known) from **Decision** (what is computed) from **Delivery** (what changes the world).
- A **risk-driven, three-profile scale** (Lite / Standard / Strict) so the amount of physical structure is proportional to risk, not dogma.
- A **zero-mocks testing posture** for the Decision / Engine layer, with observable-behavior test doubles allowed for Delivery.
- **Injection into SDD and ODD** as architectural criteria inside the existing phases of either workflow — DDS adds **no new phases** to SDD or ODD. Both methodologies remain fully supported and coexist.

### What DDS does NOT claim yet
- It does **not** claim proven, universal reductions in token cost, AI exploration cost, regression rate, or refactor blast radius.
- It does **not** claim the profiles are optimal prescriptions — they are **recommendations to test**, ranked by observed risk rather than by measured outcomes.
- It does **not** claim Zero-Mocks is universally superior to controlled mocks; it is only the *proposed* posture for pure Engines, to be validated empirically.
- It does **not** claim DDS replaces OOP or Clean Architecture as a general rule. It proposes DDS as a *candidate* better suited to agentic development, to be compared against alternatives.

### What must be measured
| Dimension | Metric to capture | Hypothesis to test |
| :--- | :--- | :--- |
| Token / context cost | Tokens read & written per task | DDS *reduces* the tokens an agent must load. |
| Files explored | Number of files opened per change | Slice isolation *bounds* exploration to one slice. |
| Test runtime | Engine unit-test suite duration | Zero-Mocks Engines run *in milliseconds*. |
| Refactor blast radius | Files touched per behavior change | Slice isolation *contains* the blast radius. |
| Regression rate | Defects introduced per change | Fewer *cross-slice* regressions. |
| Setup overhead | Time / files to start a new slice | Profiles keep startup *proportional to risk*. |
| Cognitive overhead | Reviewer effort per change (heuristic) | Linear slices are *cheaper to review*. |
| Profile promotion | Frequency of Lite → Standard → Strict moves | Promotions track *real pain*, not fashion. |

### How claims will be validated or falsified
Each future **worked example** and **benchmark** will publish its raw measurements against the table above. A hypothesis is **supported** only when repeated examples reproduce the expected direction and magnitude; it is **falsified** when examples contradict it, and the corresponding recommendation is then **downgraded or removed** from DDS.

> The Spanish reference standard mirrors this framing: see [Prospecto Medible](docs/es/DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar).

> 🗺️ **What's defined vs. what's missing:** the [DDS Operational Roadmap](docs/es/DDS_Roadmap_Operativo.md) lists what DDS + DoD already define and what remains to make the method more robust (promotion/demotion playbook, initial measurable thresholds, a real DDS Lite CRUD example, a Lite → Standard → Strict evolution, an operational definition of refactor blast radius, a persistence-by-profile guide, a public contract evolution manual, and the consolidating DDS operational playbook).

---

## Why This Method? (The AI Paradigm Shift)

Traditional software architectures (e.g., Hexagonal, Clean Architecture, DI-heavy Clean DDD) were designed to accommodate human cognitive limitations by wrapping data and behavior inside abstract, state-hiding objects. 

However, in the era of **Agentic AI Coding**, these patterns are *hypothesized* to introduce significant friction:
1. **Context Window Inflation (File Jumping):** To understand or implement a single business feature, an AI agent may need to jump across dozens of files (interfaces, services, DTOs, mappers, repositories). This is *expected to* bloat the token context, degrade reasoning accuracy, and increase LLM costs.
2. **Fragile Testing Suites (Mocking Hell):** Testing service layers with dynamic mock setups (such as Moq or NSubstitute) couples tests to internal code structures. When an AI refactors code, these mocks tend to break, which can cause the agent to get stuck in self-repair loops.
3. **Implicit Side Effects:** Mixing network I/O or database transactions inside logical validation blocks tends to make the code harder for an AI to analyze statically, which may lead to code hallucinations and regressions.

**DDS proposes to address this by designing software around isolated data flows, making it a *candidate* harness for agentic development — to be validated through examples and benchmarks, not assumed.**

---

## Proposed Benefits for AI-Agentic Development

> The items below are **hypothesized benefits** of the DDS method, to be validated through examples and benchmarks — not established outcomes. See [the prospectus](#dds-as-a-measurable-prospectus-not-a-proven-framework).

### 1. Granular Context Segmentation
By dividing the application into vertical, self-contained **Slices** (organized into `contracts.XX`, `engine.XX`, and `delivery.XX`), we *aim to* bound the AI's reading scope. An AI agent modifying a specific business flow is *expected to* only need to read and write files within that single Slice directory, so it does not need to load the rest of the application context — *intended to* keep accuracy high.

### 2. Compile-Time Customs Guard (DDS Strict)
DDS uses the compiler as a strict physical boundary **in the Strict profile**. By compiling Slices as independent binary assemblies (e.g., separate C# `.csproj` libraries or C++ static libs) and using `internal` visibility, the compiler prevents the AI from leaking implementation details across slices. If the AI violates the contract boundaries, the compilation breaks immediately, providing instant, deterministic feedback. In Lite and Standard, the same isolation is achieved by **convention and file layout** until risk justifies the compiler.

### 3. Zero-Mocks Determinism
Since business logic is isolated in pure static **Engines** (Decision) that receive and return plain value-type structs (PODs), testing becomes a pure mathematical exercise ($f(x) = y$). The AI is *expected to* be able to write and run unit tests in milliseconds without setting up complex mock engines, dynamic proxies, or database connections. If a test fails, the AI should be able to locate the logical error quickly by comparing flat input and output data.

> **Refined scope:** Dynamic mocks are prohibited for **Decision / Engine** tests. Delivery / API / DB tests may still use controlled test doubles, fakes, testcontainers, or fake HTTP servers when they validate **observable behavior** (not internal call choreography). See [`DDS_TDD_Sin_Mocks.md`](docs/es/DDS_TDD_Sin_Mocks.md).

---

## DDS Inside SDD and ODD (No Extra Steps)

DDS is **workflow-agnostic**: it does not modify the flow of the host methodology nor add phases. It is injected as **architectural criteria** inside the existing phases of either agentic workflow, after the idea is defined. SDD (Spec-Driven Development) and ODD (Organic Driven Development) are both supported and coexist — choose whichever the project already uses; DDS adapts to it.

### Injection into SDD (Spec-Driven Development)

| SDD phase | What DDS adds |
| :--- | :--- |
| **Design / Spec planning** | Choose the **profile** (Lite / Standard / Strict), define the **slice boundary** and its name, fix the **contract shape**, and pick the **TDD strategy**. |
| **Apply** | Implement using the selected profile and keep the `Data / Decision / Delivery` separation **proportionate** to the slice's risk. |
| **Verify** | Check tests, boundaries, blast radius, and confirm that the **profile choice was proportional** to the actual risk. |

### Injection into ODD (Organic Driven Development)

| ODD phase | What DDS adds |
| :--- | :--- |
| **Authorize / Explore** | Read-only phase: bound the exploration to the target **slice** (read only the public `contracts.XX` of neighbor slices). No structural commitment yet. |
| **Resolve uncertainty / Classify / Track** | Choose the **profile** (Lite / Standard / Strict) based on observed risk, define the **slice boundary** and its name, fix the **contract shape**, and record the choice in the feature task document. |
| **Implement task by task** | Implement with the selected profile, keep the `Data / Decision / Delivery` separation **proportionate** to the slice's risk, and honor the context-bounding rules of the target slice. |
| **Check / Close** | Same as SDD Verify: check tests, boundaries, blast radius, and confirm that the **profile choice was proportional** to the actual risk. Each task closes with a reviewable work-unit commit. |

> The SDD and ODD mappings are equivalent: **planning picks the profile, implementation keeps the separation, verification confirms proportionality.** Only the phase names differ.

### Slice Naming & Sizing

A slice represents a **reason for change / capability**, not necessarily a table or entity.

| Context | Typical slice granularity |
| :--- | :--- |
| Simple CRUD | `ProductManagement`, `CustomerManagement` are valid. |
| Business domains | `Inventory`, `Sales`, `Billing` are usually better than `Customer` / `Product` / `Order` when those entities are just shared data inside a larger capability. |
| Very complex flows | Sub-slices / use-case slices such as `ApproveRefund`, `CancelSale` can be justified. |

> Entity-per-slice is **not automatically wrong** — it is just **not automatic**. Size the slice by capability and blast radius, not by table count.

---

## Documentation Index (Spanish)

All detailed architectural standards, refactoring playbooks, and embedded implementations are located under the `docs/es/` directory:

| Document | Description |
| :--- | :--- |
| 📘 **[DDS Work Method Standard](docs/es/DDS_Metodo_Trabajo_IA.md)** | The reference standard for DDS + DoD in C++20 and C# (.NET 8+). Covers visibilities, namespaces, and transaction management. |
| 📙 **[Introduction to Data-Oriented Design (DoD)](docs/es/DDS_Introduccion_DoD.md)** | Conceptual primer explaining the shift from OOP to DoD. Discusses CPU/RAM latency (Memory Wall), cache line spatial locality, and AoS vs. SoA layouts. |
| 📝 **[Step-by-Step Refactoring Guide](docs/es/DDS_De_POO_a_Slices.md)** | Practical walkthrough taking a standard OOP `OrderService` (with DI, interfaces, and DbContext) and refactoring it into a clean, decoupled DDS Slice. |
| 🧪 **[Zero-Mocks Unit Testing Standard](docs/es/DDS_TDD_Sin_Mocks.md)** | Technical guide for building ultra-fast, deterministic test suites on pure Engines using stack-allocated structs without mocking frameworks. |
| 💾 **[Segregated Persistence Design](docs/es/DDS_Persistencia_Segregada.md)** | Deep dive into the architectural trade-offs of splitting database contexts per Slice, discussing database coupling vs. DRY principles. |
| ⚙️ **[DDS in Embedded Systems](docs/es/DDS_Sistemas_Embebidos.md)** | Architecture guide for implementing DDS on resource-constrained chips (ESP8266/ESP32) with zero heap allocation, setup isolation, and ISR data flow. |
| 🗺️ **[DDS Operational Roadmap](docs/es/DDS_Roadmap_Operativo.md)** | What is already defined for DDS + DoD and what remains to make the method more robust (promotion/demotion playbook, measurable thresholds, worked examples, contract evolution manual). Spanish. |
| 🛠️ **[DDS CLI Tool Decision Record](docs/es/DDS_Tool_CLI_Decisiones.md)** | Design decisions for the companion Go CLI (`tools/dds-guard/`, planned): boundary guard (before/after) + prompt/skill factory, both driven by the criterion as structured data. Spanish. |
| 📜 **[Slice Charter + Architecture Tests](docs/es/DDS_Slice_Charter.md)** | Per-slice intent doc (`SLICE.md`: purpose, non-goals, language — no code-dependent docs) plus executable boundary tests that run with the suite. Spanish. |
| 🧭 **[Operational Workflows + Decision Matrix](docs/es/DDS_Workflows.md)** | How a change enters the system (decision matrix D-1..D-7), how to modify an existing feature's behavior, how to birth a new slice, multi-slice changes, and the agent-facing operational rule. Spanish. |
| 🗂️ **[Use-Case Discussion Register](docs/es/DDS_Casos_Discusion.md)** | WORKING REGISTER (not settled criterion): the slice ≠ program correction (host/composition root) and the C1–C15 case catalog with discussion status; each resolved case graduates into the workflows doc. Spanish. |

---

## Repository Structure

```text
/data-driven-slices
├── README.md                 <-- This entry point
└── docs
    └── es                    <-- Spanish documentation directory (actively expanding)
        ├── DDS_De_POO_a_Slices.md
        ├── DDS_Casos_Discusion.md
        ├── DDS_Introduccion_DoD.md
        ├── DDS_Metodo_Trabajo_IA.md
        ├── DDS_Persistencia_Segregada.md
        ├── DDS_Roadmap_Operativo.md
        ├── DDS_Sistemas_Embebidos.md
        ├── DDS_Slice_Charter.md
        ├── DDS_Tool_CLI_Decisiones.md
        ├── DDS_TDD_Sin_Mocks.md
        └── DDS_Workflows.md
```

> 🛠️ **Planned:** the companion Go CLI (`tools/dds-guard/`) will live in this repository once implemented — see the [CLI Tool Decision Record](docs/es/DDS_Tool_CLI_Decisiones.md).
