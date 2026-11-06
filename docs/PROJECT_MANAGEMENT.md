---
title: stm8-rs Project Management and Contribution Workflow
version: 1.1
status: active
roadmap: roadmap/Rust_for_STM8_SDCC_Feasibility_and_Roadmap_v1.0.md
---

# stm8-rs Project Management and Contribution Workflow

This document defines the initial Kanban workflow for `stm8-rs`. It is intended to be readable by both human contributors and coding agents.

The architectural and validation requirements are defined in:

- [Feasibility and roadmap v1.0 (Markdown)](roadmap/Rust_for_STM8_SDCC_Feasibility_and_Roadmap_v1.0.md)

If this document and the roadmap disagree, the roadmap governs technical correctness, safety, and gate criteria. This document governs task organization and contribution mechanics.

## 1. Project principles

Preserve these principles in every task and review:

1. **Memory safety without hardware compromise.** Improve correctness without increasing runtime hardware-resource requirements unnecessarily.
2. **Pragmatism over abstraction.** Standard abstractions are preferred when affordable; smaller specialized implementations are acceptable when measurements favor them.
3. **Correctness is non-negotiable.** Generality, ergonomics, and optional features may be reduced instead of weakening Rust semantics, memory safety, volatile behavior, ABI rules, or observable functionality.
4. **Evidence over assumption.** Generated assembly, linker maps, resource measurements, simulator results, and hardware traces establish success.
5. **Unsafe implementation is not automatically unsafe behavior.** Compact unsafe code is permitted only behind an audited interface with explicit invariants and tests.

## 2. GitHub Projects setup

Create one GitHub Project for the repository. Use a Kanban board as the primary view and table views for gate and validation reports.

### 2.1 Board columns

| Column | Meaning | Entry requirement |
|---|---|---|
| Backlog | Unrefined future work | A problem or objective is identified |
| Ready | Actionable task | Scope, acceptance criteria, artifacts, gate, and validation are defined |
| In progress | Actively being implemented | An owner is assigned; related blockers are understood |
| Review / CI | Implementation awaits verification | Tests/builds and requested review are running |
| Blocked | Work cannot proceed | Blocker and unblocking condition are recorded |
| Done | Completed and reproducible | Definition of Done is satisfied |

Limit work in progress deliberately. A contributor should normally have one primary implementation task and at most one additional research or validation task.

### 2.2 Status transitions

Normal flow:

```text
Backlog → Ready → In progress → Review / CI → Done
                         ↘ Blocked → Ready

Physical hardware validation is a separate requirement tracked on the issue, not an alternative completion status.
```

Rules:

- Do not move an issue to **Done** solely because code compiles.
- Return issues to **In progress** when review or testing reveals incorrect behavior.
- Do not keep **Blocked** as a permanent parking lot. Record a research task, owner, and next review date.
- A hardware failure produces a minimized reproduction before a fix is attempted.
- An issue can remain in **Review / CI** while required hardware evidence is collected; no hardware-required issue reaches **Done** without that evidence.
- Start F0 with one **In progress** task and approximately three **Ready** tasks; leave the remaining workstreams in **Backlog** until refined.

## 3. Milestones and gates

Milestones represent evidence gates, not large single issues.

| Gate | Main objective | Required evidence |
|---|---|---|
| F0 | Lead compiler feasibility on STM8S103F3P6 | Target model, MIR→C→SDCC pipeline, semantic tests, `core` blocker inventory, uCsim and lead-board execution, initial resource reports |
| F1 | Mixed C/Rust migration feasibility on lead MCU | Bidirectional ABI, interrupt entry, priority-preserving critical sections, helper reentrancy, GPIO/timer/UART examples |
| F2 | STM8S003F3P6 compatibility qualification | Independently checked device manifest, board configuration, applicable semantic/SPL tests, hardware and resource reports |
| Release | Documented stable subset | Feature contract, both device qualifications, regression and SPL-derived suites, reproducibility, resource and latency reports |

Do not decompose the entire roadmap into hundreds of issues. Keep the 17 F0 identifiers as stable workstream or parent identifiers, but refine only the next few executable child issues. Refine F1 after F0 results clarify the architecture.

### 3.1 F0 critical path and parallel risk work

The primary dependency chain is:

```text
F0-001 (pinned toolchain)
    |
    +--> F0-003 (target contract) ---+
    |                                +--> F0-005 (first end-to-end executable)
    +--> F0-004 (MIR extraction) ----+               |
                                                    v
                            F0-006 / F0-008 / F0-009 (semantic validation)
                                                    |
                                                    v
                                           F0-017 (gate review)
```

Run **F0-002** (C hardware baseline), **F0-010** (`core` blockers), and **F0-013** (resource-report tooling) as early parallel risk-reduction work. The diagram is a planning dependency sketch, not a substitute for individual issue dependencies. F0-007 and other required semantic cases must also be completed before F0 passes.

**First board configuration:** F0-001 In progress; F0-002, F0-003 and F0-004 Ready; all other F0 workstreams in Backlog. Reprioritize based on actual blockers rather than filling the board with speculative Ready items.

## 4. Project fields

Configure these fields in GitHub Projects:

| Field | Type | Values or format |
|---|---|---|
| Gate | Single select | F0, F1, F2, Release, Later |
| Area | Single select | rustc, MIR-C, core, ABI, SDCC, runtime, PAC-HAL, SPL, tooling, docs |
| Kind | Single select | Research, Implementation, Test, Review, Documentation |
| Validation required | Multi-select or checklist | Host, uCsim, STM8S103, STM8S003, Commercial compiler, Not required |
| Validation evidence | Text or issue-linked checklist | For each required environment: Pending, Passed, Failed, Blocked; link to run artifacts |
| Effort | Single select | XS (<2h), S (2–8h), M (1–3d), L (>3d; split if possible) |
| Risk | Single select | Normal, Wrong-code, ABI, Interrupt, Resource, Toolchain |
| Artifacts | Text | Required evidence and artifact index/run IDs, separated by semicolons |
| Blocked by | Text or issue link | Blocking issue/reason and unblocking condition |
| Owner | Assignee | Responsible contributor |

## 5. Labels

Use a small, consistent label vocabulary:

```text
area:rustc       area:mir-c       area:core        area:abi
area:sdcc        area:runtime     area:pac-hal     area:spl
area:tooling     area:docs
kind:research    kind:implementation
kind:test        kind:review      kind:docs
hardware:ucsim   hardware:stm8s103 hardware:stm8s003
risk:wrong-code  risk:abi         risk:interrupt   risk:resource
good first issue help wanted      blocked
```

Do not use labels as a substitute for acceptance criteria or required artifacts.

## 6. Task decomposition rules

A ready issue must:

- have one verifiable outcome;
- identify its gate, area, validation, and dependencies;
- state expected behavior and explicit non-goals;
- name required artifacts;
- preserve correctness and resource requirements;
- normally require less than three days of work (larger items become parent issues or are divided into child issues);
- be independently reviewable where practical.

Split tasks when they contain independent implementation and validation work. Keep hardware-specific validation visible rather than hiding it inside a large “compiler” issue.

Do not create tasks whose only acceptance criterion is “it builds,” “looks correct,” or “agent says it works.”

The F0 backlog identifiers are **workstreams**, not promises that every row fits in one small implementation issue. In particular, split F0-005 into pipeline stages, F0-006 by operation/width, F0-008 by aggregate kind, and F0-015 into GPIO, timer and UART vertical slices. F0-010 is a bounded **investigation and blocker inventory**, not an obligation to complete `core` during that issue. Keep stable IDs and link smaller child issues to their parent.

A feature is not complete merely because its code has landed. Previously passing semantic regression cases must continue to pass, and new failures must be minimized and explained before changing either the implementation or its oracle.

## 7. Initial F0 backlog

The IDs below are stable **parent/workstream** identifiers. Use them in issue titles and artifacts; create small linked child issues as each workstream becomes actionable. Order the child issues along the critical path rather than scheduling all workstreams at once.

| ID | Task | Minimum acceptance evidence |
|---|---|---|
| F0-001 | Reproducible repository and toolchain baseline | Pinned rustc/Cargo/SDCC configuration, clean-build instructions, CI logs, artifact directories |
| F0-002 | Direct-register C hardware baseline | SDCC C binary, startup/linker files, SWIM programming procedure, observed lead-board behavior |
| F0-003 | Initial STM8 medium-model target specification | Target JSON and schema version; endian, pointer, integer, alignment, and atomic contract notes; layout assertions |
| F0-004 | Minimal target-aware MIR extraction | MIR for small standalone functions, rustc revision, extraction commands, layout diagnostics |
| F0-005 | First MIR→C→SDCC executable | Minimal function and return path, generated C, SDCC assembly, uCsim result, lead-board execution |
| F0-006 | Integer arithmetic and control flow | Signed/unsigned 8/16/32-bit boundary corpus, checked and wrapping behavior, branches and loops, differential results |
| F0-007 | Shift/division edge cases and helper inventory | `MIN / -1`, remainder, shift counts, division by zero, emitted helpers, reentrancy notes |
| F0-008 | References, arrays, structs, and tagged enums | Target-layout assertions, offset/discriminant tests, generated C and assembly, validity cases |
| F0-009 | Volatile MMIO lowering | Exact read/write traces for constant and computed addresses, no lost/reordered observable operations |
| F0-010 | `core` bootstrap investigation | `build-std=core` attempt, categorized blockers, intrinsic/compiler-builtins inventory, credible remediation plan |
| F0-011 | Fixed array and flash-storage lowering | `&[u8; N]` cases, literal/reference placement, startup-copy analysis, no terminator requirement |
| F0-012 | `&str`, `&CStr`, and `StaticString<N>` benchmarks | Length/ABI tests, embedded/trailing NUL behavior, non-UTF-8 data, constructor validation, whole-image comparison |
| F0-013 | Resource-report tooling | Occupied-flash calculation, `.data`/`.bss`, reserved stack, high-water protocol, assembly/map artifacts |
| F0-014 | SDCC optimization matrix | Selected optimization profiles, semantic results, code/stack/cycle changes, minimized regressions |
| F0-015 | Initial GPIO/timer/UART vertical slices | Observable lead-board behavior, equivalent direct-register C baselines, resource and timing reports |
| F0-016 | Optional time-boxed AVR reference experiment | `avr-none` target/MIR/core/helper notes and reusable semantic cases; not an F0 completion requirement |
| F0-017 | F0 gate review and go/no-go report | All required evidence indexed, failures and blockers disclosed, explicit go/no-go/redesign decision |

## 8. Issue template

Use this structure for implementation and research issues:

```markdown
## Objective

<One verifiable outcome.>

## Gate and classification

- Gate: F0 / F1 / F2 / Release / Later
- Area: rustc / MIR-C / core / ABI / SDCC / runtime / PAC-HAL / SPL / tooling / docs
- Kind: research / implementation / test / review / documentation
- Validation: host / uCsim / STM8S103 / STM8S003 / commercial compiler / not required
- Effort: XS / S / M / L
- Risk: normal / wrong-code / ABI / interrupt / resource / toolchain

## Dependencies

<Issue links, hardware, documents, or “none”.>

## Acceptance criteria

- [ ] <Observable behavioral requirement>
- [ ] <Semantic or safety requirement>
- [ ] <Resource or performance requirement, if applicable>
- [ ] <Required test passes>

## Required artifacts

- Reproducibility run ID and link to `manifest.json`
- Rust source and revision
- MIR, generated C, and assembly, when applicable
- Toolchain names, revisions, and flags
- Map files and resource reports, when applicable
- Simulator or hardware evidence required by the issue

## Out of scope

<Explicitly excluded work.>

## Completion notes

<Final measurements, remaining limitations, regression results, and follow-up issue links.>
```

## 9. Example ready issue

```markdown
Title: F0-006: Lower u16::wrapping_add without C signed-overflow UB

Objective:
Implement and validate u16 wrapping addition for STM8 MIR→C lowering.

Gate and classification:
- Gate: F0
- Area: MIR-C
- Kind: implementation + test
- Validation: host + uCsim
- Effort: S
- Risk: wrong-code

Acceptance criteria:
- [ ] 0x0000, 0x0001, 0x7fff, 0x8000, 0xfffe, and 0xffff boundary cases pass.
- [ ] Generated C does not invoke signed overflow for valid Rust inputs.
- [ ] Results match target-independent native Rust arithmetic expectations.
- [ ] Assembly and helper calls are reviewed for size and reentrancy.

Required artifacts:
- Rust corpus and expected outputs
- MIR and generated C for minimized cases
- SDCC assembly and command-line flags
- Host differential and uCsim results
- Run ID, artifact manifest and baseline regression results

Out of scope:
- u32 support and interrupt-entry shims
```

## 10. Reproducible evidence and Definition of Done

### 10.1 Test-run identity and artifact manifest

Every compiler-relevant experiment must have a unique run ID (for example, `F0-006/run-0001`) and a machine-readable artifact manifest. A minimal layout is:

```text
artifacts/F0-006/run-0001/
  manifest.json
  source.rs
  output.mir
  output.c
  output.asm
  firmware.ihx
  results.json
```

Only include files actually generated by the run. `manifest.json` records exact rustc/Cargo, MIR-to-C backend and SDCC revisions, MCU/board manifest revisions, compiler and linker flags, source hashes, test oracle revision, executed test environments, measurement methods, artifact hashes and tool exit status. `results.json` records expected versus observed behavior, test pass/fail/blocked states and resource measurements with units. A failed run is still retained and identifiable. Do not fabricate hardware results or an artifact solely to populate the directory template.

The test oracle must be independently derived from Rust semantics, hardware documentation, or validated reference behavior; generated C output cannot be its own oracle. Retain the previous passing corpus as a regression baseline.

### 10.2 Definition of Done

An issue is Done only when:

- all acceptance criteria pass or an approved scope change is recorded;
- generated behavior matches the issue’s functional contract;
- required tests are present in the regression suite;
- required artifacts are linked or attached with run IDs and enough provenance to reproduce the result;
- all previously established applicable semantic regression tests still pass, with changes to expected outputs independently justified;
- unsafe invariants and unsupported cases are documented;
- resource effects are reported where relevant;
- another contributor or an independent review pass has examined the change;
- no known wrong-code defect remains hidden or renamed as expected behavior.

A failed experiment is complete when its evidence and conclusion are documented. It does not count as a passed feature gate.

## 11. Agent contribution rules

Coding agents must:

1. Read the roadmap and the issue before editing.
2. Keep changes within the issue’s stated scope.
3. Use the project’s pinned toolchain and actual build/test commands.
4. Preserve independent expected outputs and safety tests.
5. Never weaken, delete, or rewrite a failing test merely to obtain a pass.
6. Minimize and archive failures before attempting fixes.
7. Run the established semantic regression corpus after compiler changes; never silently weaken existing guarantees.
8. Keep test-oracle changes independent from implementation changes, and justify each expected-output change against an external contract.
9. Inspect generated C and assembly when semantics or optimization are involved.
10. Mark unsupported constructs explicitly instead of approximating them.
11. Never claim simulator or hardware validation that was not actually run.
12. Report unverified work and remaining blockers plainly.
13. Keep third-party licenses and proprietary artifacts out of project-owned MIT/Apache-2.0 files.
14. Avoid committing credentials, compiler license files, activation data, or proprietary installers.

Human contributors remain responsible for architecture, unsafe-code review, hardware results, and gate decisions.

## 12. CI and validation policy

Run on public GitHub-hosted CI where practical:

- clean pinned-toolchain builds;
- host semantic differential tests;
- generated-C compilation;
- deterministic uCsim tests;
- artifact upload for MIR, C, assembly, maps, run manifests and reports;
- regression checks against the previously passing semantic corpus.

Keep these activities separate and manually controlled:

- physical-board programming and measurements;
- oscilloscope or logic-analyzer capture;
- proprietary compiler collection;
- licensed self-hosted Windows machines.

Do not expose licensed self-hosted infrastructure to untrusted pull-request code. Proprietary compilers are optional comparison tools and are not required for mainline reproducibility.

## 13. Contributor onboarding

Good first issues should be small and evidence-oriented, for example:

- add one arithmetic boundary family and minimized expected outputs;
- port and validate one SPL example’s C baseline;
- compare one generated function with equivalent direct-register C;
- extend a device-manifest field against a cited datasheet table;
- reproduce one documented SDCC helper or ABI test;
- improve artifact indexing or resource-report generation.

Every issue should make clear whether compiler expertise, embedded experience, physical hardware, or only a host computer is required.

## 14. Maintenance cadence

On a recurring basis:

- review Blocked items and unblock or close them;
- keep Ready tasks small and current;
- review wrong-code and interrupt risks first;
- publish F0/F1/F2 evidence summaries without hiding failures;
- create follow-up issues instead of expanding completed ones;
- refine the next gate only when current evidence supports it.

The objective is steady, inspectable progress—not a large backlog with uncertain status.
