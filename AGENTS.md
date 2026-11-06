# Agent instructions for stm8-rs

These instructions apply to the entire repository. They bind coding agents and other contributors to the project's existing architecture and evidence policy; they do not replace the detailed documents linked below.

## 1. Read the governing documents first

Before editing implementation or validation files, read:

1. the applicable issue, including its scope, acceptance criteria, dependencies, and required artifacts;
2. [the feasibility and architecture roadmap](docs/roadmap/Rust_for_STM8_SDCC_Feasibility_and_Roadmap_v1.0.md), especially the sections relevant to the task;
3. [the project management and contribution workflow](docs/PROJECT_MANAGEMENT.md).

For documentation work, also read [the documentation policy](docs/DOCUMENTATION.md). For vendor material or third-party code, read [the reference policy](docs/references/README.md) and [THIRD_PARTY.md](THIRD_PARTY.md).

Authority is divided as follows:

- The roadmap governs technical correctness, safety, supported behavior, resource criteria, and gate decisions.
- The project-management document governs task decomposition, review, evidence, and completion.
- The documentation and reference policies govern their respective files.
- If documents conflict or a request would violate them, do not silently choose a convenient interpretation. Record the conflict and keep technical changes blocked until the governing document or task scope is explicitly revised.

When changing behavior, policy, commands, or architecture described in this file or another project document, update that documentation in the same change.

## 2. Represent the project honestly

- The repository is currently at **F0 feasibility and planning**. It does not yet contain a working compiler, installable toolchain, build procedure, or validated device support.
- Treat the roadmap as a specification and validation plan, not as evidence of implemented functionality.
- Never describe a planned, experimental, deferred, or partially validated feature as supported. A device or feature is supported only after its corresponding evidence gate passes.
- Do not invent build, test, installation, simulator, or hardware commands. Once a pinned toolchain exists, use its actual documented commands and flags.
- Do not claim success merely because code compiles, imports, or looks correct. State precisely what was run, what passed, what failed, and what remains unverified.
- A documented failed experiment may complete a research task, but it never counts as a passed capability gate.

## 3. Keep work scoped and reviewable

- Identify one verifiable outcome, the applicable gate/workstream, explicit non-goals, required validation, and artifacts before starting. If no issue exists, derive these from the user's request without expanding its scope.
- Preserve stable F0 workstream identifiers and dependencies. Split implementation and independent validation into reviewable child tasks when necessary; do not decompose the entire roadmap speculatively.
- Keep F0 ordered around its documented critical path. Do not begin broad F1/F2, `core`, PAC/HAL, or ecosystem expansion ahead of the evidence that the roadmap requires.
- Do not fix unrelated defects or rewrite adjacent systems. Report unrelated findings separately.
- Record blockers and their unblocking conditions. Do not disguise blocked validation as a pass or rename incorrect behavior as expected behavior.

## 4. Preserve the architecture and runtime contract

The required initial path is:

```text
no_std Rust -> rustc frontend and MIR -> target-aware MIR-to-C lowering
            -> SDCC-compatible C and thin runtime/shims -> SDCC STM8 backend
            -> firmware image
```

- Reuse rustc parsing, type checking, borrow checking, traits, generics, and monomorphization. Do not substitute a syntax-only translator or reimplement the Rust frontend.
- Reuse SDCC for STM8 instruction selection, register allocation, assembly, and linking. Do not switch to an LLVM/native-backend design without an explicit roadmap revision.
- Keep startup, interrupt, and ABI C shims thin and audited; they must not become a hidden implementation of application logic.
- Start only with the exact lead part **STM8S103F3P6**. Qualify **STM8S003F3P6** independently at F2; never infer support from a similar name.
- Use the SDCC **medium memory model exclusively** for initial experiments. Do not mix memory models. Verify its 16-bit object/function addresses, big-endian integer storage, alignment, and `__sdcccall(1)` ABI from actual output rather than host assumptions.
- Keep architecture target data, exact-MCU manifests, and board configuration separate. Check physical markings, BOM, clocks, wiring, and instrumentation before making board-specific claims.
- Preserve the initial restricted runtime: `no_std`, no allocation, `panic=abort`, one execution core, and explicit unsupported-feature diagnostics.
- Follow roadmap Sections 3 and 5 as the authoritative feature matrix. Reject unsupported constructs clearly instead of approximating their behavior. In particular, do not silently introduce allocation, unwinding, threading, deferred numeric types, trait objects, recursion, inline assembly, or unsafe `static mut` conveniences.
- Prefer incremental migration of validated C firmware and direct-register/SPL comparisons. Do not claim a safe PAC/HAL until compiler lowering, ownership, aliasing, synchronization, and hardware behavior have been validated.

## 5. Make correctness and safety non-negotiable

Reduce generality, ergonomics, or supported functionality rather than weakening any rule below.

- Generated C must not introduce C undefined behavior for valid Rust inputs. Handle signed overflow, wrapping and saturating operations, casts, shifts, division/remainder, and signed `MIN / -1` according to Rust semantics in every supported build mode.
- Compute and assert target layouts. `repr(Rust)` is not C layout; matching byte size alone does not validate enums, niches, padding, references, validity, provenance, or aliasing. `repr(C)` alone does not establish SDCC ABI compatibility or provide C bitfields.
- Preserve volatile MMIO operations exactly: no lost, duplicated, added, or improperly reordered observable accesses. Inspect generated C and assembly for constant and computed addresses.
- Validate SDCC calling conventions, arguments, returns, aggregates, pointers, linkage, and reciprocal C/Rust calls with emitted code and runtime evidence. Host Rust is never an STM8 layout, pointer-width, endianness, address-space, timing, or ABI oracle.
- Keep `unsafe` small, explicit, and behind an audited interface with documented invariants and tests. An `&mut self` API, type-state wrapper, or zero-sized token alone does not prove safety.
- A safe peripheral API must validate addresses and masks, prevent forging and aliasing, account for port-wide ownership, and provide interrupt-safe read-modify-write behavior.
- Audit every emitted or linked arithmetic/runtime helper for stack use and reentrancy in main and ISR contexts. Define and test panic-in-ISR behavior.
- Do not use SDCC `__critical` blindly for Rust critical sections. Save, mask, and restore the original CC state, including condition flags and interrupt priorities; test nesting, early returns, ISR transitions, and actual pinned-SDCC assembly.
- Use standard Rust arrays, slices, `&str`, and `&CStr` where validated. Preserve bounds, UTF-8, embedded-NUL, and trailing-NUL semantics; do not replace length-based operations with unbounded C string functions. Add a custom string abstraction only for a measured requirement and validate its whole-image cost.

## 6. Preserve independent tests and failure evidence

- Derive expected behavior independently from Rust semantics, hardware documentation, or validated reference behavior. Generated C must never be its own oracle.
- Keep the previously passing semantic corpus as a regression baseline. Run all applicable established regressions after compiler changes.
- Never weaken, delete, skip, or rewrite a failing test merely to obtain a pass. Keep test-oracle changes independent from implementation changes and justify every expected-output change against an external contract.
- Minimize and archive a compiler or hardware failure before attempting a fix. Preserve failed runs and their provenance.
- Exercise boundary values, invalid/unsupported cases, optimization profiles, and applicable ABI, interrupt, volatile, layout, and validity cases—not only happy paths.
- When semantics or optimization change, inspect and retain relevant MIR, generated C, assembly, map files, and helper calls. A host-only result cannot substitute for required uCsim or physical-hardware evidence.
- Treat disassembly and binary identity as diagnostics, not correctness oracles. Compare observable behavior independently.

## 7. Produce reproducible validation evidence

Every compiler-relevant experiment must have a unique run ID and a machine-readable manifest under the established `artifacts/<workstream>/<run-id>/` convention.

- Include only files actually produced. The manifest must identify exact tool and backend revisions, MCU/board revisions, flags, source and artifact hashes, oracle revision, environments, measurement methods, and tool exit status.
- Record expected versus observed behavior, pass/fail/blocked states, and resource measurements with units in the results data. Never fabricate an artifact, run, simulator result, or hardware trace.
- Use the pinned rustc/Cargo/rust-src and SDCC toolchain. Do not silently upgrade tools, alter flags, or substitute another compiler. Proprietary compilers are optional comparisons, never mainline dependencies or semantic oracles.
- Run physical programming and measurements only through the controlled local procedure. Do not expose licensed infrastructure to untrusted pull-request code.
- Report occupied flash address bytes rather than HEX text length; report `.data + .bss`, reserved stack, total RAM, measured stack high-water, and conservative call-graph/ISR bounds separately and with units.
- Measure cycles and interrupt latency with defined events, calibrated methods, and disclosed overhead. Compare equivalent behavior per workload against the required direct-register C or SPL C baseline; aggregates must not hide individual failures.
- Apply the roadmap's F0/F1 stop thresholds and release resource-parity rules exactly. Zero-cost abstraction and C parity are hypotheses to measure, not assumptions to assert.

Do not commit transient local build output, ignored experiment results, credentials, or proprietary material. Reviewed regression fixtures are the intentional exception defined by `.gitignore`.

## 8. Apply the Definition of Done

Do not mark an issue or requested change complete until:

- every acceptance criterion passes or an approved scope change is recorded;
- observable behavior matches its contract and all required regression tests are present and passing;
- all required artifacts are linked with run IDs and sufficient provenance to reproduce them;
- applicable previous semantic regressions still pass;
- unsafe invariants, unsupported cases, resource effects, failures, and limitations are documented;
- required simulator and physical-hardware evidence actually exists;
- an independent review pass has examined the change; and
- no known wrong-code defect is hidden or reclassified as expected behavior.

Compilation alone is never sufficient for compiler work. Agents must not unilaterally declare an unsafe interface audited, a hardware gate passed, or a device qualified. If a required check cannot be run, leave the corresponding validation pending and report the blocker plainly.

## 9. Maintain documentation and third-party boundaries

- Keep `README.md` focused on status, overview, and navigation. Put substantial topic-specific material in focused Markdown documents under `docs/`.
- Use stable, descriptive, unversioned filenames for living documents and relative links to canonical files. Do not create routine versioned copies or use local filesystem paths in repository documentation.
- When restructuring documentation, preserve one canonical location, update all internal links and frontmatter paths, and verify that relative links resolve.
- Markdown is canonical. Regenerate generated counterparts such as PDFs in the same change, or explicitly record why they were not regenerated.
- Original project content is dual-licensed MIT OR Apache-2.0. Third-party code, documents, headers, libraries, adaptations, and runtime outputs retain their original licenses; never relicense them as project-owned work.
- Record third-party origin, revision, license, and local modifications in `THIRD_PARTY.md`. Review licenses before reuse or redistribution.
- Do not commit vendor documents unless redistribution permission is verified and recorded. Keep local vendor PDFs in `docs/references/vendor/` and maintain their identities and lowercase SHA-256 hashes in `docs/references/manifest.toml`.
- Never commit credentials, activation data, compiler license files, proprietary installers, or restricted runtime sources. Ignore rules are not a security boundary.

## 10. Final response requirements

Finish with a concise report containing:

1. the changed behavior and files;
2. the exact verification performed and its result;
3. linked run IDs and artifacts for compiler-relevant experiments;
4. resource and regression results where applicable; and
5. every unrun required check, unsupported case, failure, or remaining blocker.

Never imply that unexecuted validation passed. Human contributors remain responsible for architecture decisions, unsafe-code review, hardware results, and F0/F1/F2/release gate decisions.
