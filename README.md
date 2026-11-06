# stm8-rs

**Exploring a memory-safe Rust toolchain for STM8 microcontrollers via SDCC.**

`stm8-rs` is an experimental compiler and embedded-systems project. Its goal is to support a documented, resource-conscious subset of `no_std` Rust on selected STM8 devices while preserving the code size and execution efficiency expected from C firmware.

The proposed design keeps the existing Rust frontend for parsing, type checking, borrow checking, traits, generics, and monomorphization. Lowered Rust operations would then be translated into C accepted by SDCC, using SDCC's mature STM8 backend for instruction selection, assembly, and linking.

> **Project status: F0 feasibility and planning.** This repository currently contains the architecture, validation strategy, and contribution workflow—not a working compiler or installable toolchain. No STM8 device or Rust feature should be considered supported until its corresponding validation gate passes.

## Goals

- Preserve Rust's compile-time safety guarantees without unnecessary runtime hardware costs.
- Establish a target-aware MIR-to-C-to-SDCC compilation path.
- Support incremental migration of suitable C firmware to Rust.
- Validate generated C and assembly, ABI behavior, resource usage, simulator results, and physical-hardware behavior.
- Compare Rust output with direct-register C and STM8 Standard Peripheral Library baselines.
- Develop PAC/HAL-style abstractions only after the underlying compiler path is shown to be feasible.

The initial lead device is the **STM8S103F3P6**. The **STM8S003F3P6** is the planned compatibility-qualification device, not an automatically supported equivalent.

## Proposed architecture

```text
Rust source
    |
    v
rustc frontend and MIR
    |
    v
Target-aware MIR-to-C lowering
    |
    v
SDCC STM8 backend
    |
    v
STM8 firmware
```

This is the intended design, not a claim that the pipeline is already implemented. The initial work uses SDCC's medium memory model and a restricted runtime; broader `core` support, additional memory models, and higher-level embedded abstractions remain validation milestones.

## Milestones

| Gate | Intended outcome |
|---|---|
| **F0** | Demonstrate lead-compiler feasibility on STM8S103F3P6, including the compilation pipeline, semantic tests, `core` blocker inventory, simulator execution, hardware evidence, and resource reports. |
| **F1** | Demonstrate mixed C/Rust migration, bidirectional ABI calls, interrupt handling, critical sections, and representative peripheral examples. |
| **F2** | Qualify STM8S003F3P6 compatibility independently. |
| **Release** | Publish a documented stable subset with reproducible builds, regression evidence, and resource measurements. |

A failed experiment may produce a useful feasibility result, but it does not count as a passed capability gate.

## Documentation

- [Feasibility, architecture, and validation roadmap](docs/roadmap/Rust_for_STM8_SDCC_Feasibility_and_Roadmap_v1.0.md) — technical requirements and decision gates.
- [Project management and contribution workflow](docs/PROJECT_MANAGEMENT.md) — issue structure, review policy, and definition of done.
- [Agent instructions](AGENTS.md) — mandatory architecture, validation, documentation, and completion rules for coding agents.
- [Documentation policy](docs/DOCUMENTATION.md) — document organization, stable filenames, internal linking, and revision conventions.
- [Vendor reference documents](docs/references/README.md) — external hardware manuals and local-copy verification.
- [Third-party notices](THIRD_PARTY.md) — ownership, licensing, and redistribution policy for external materials.

## Contributing

Read the roadmap and the contribution workflow before starting implementation. Work should have a verifiable outcome, explicit validation requirements, and reproducible evidence. Compilation alone is not sufficient for compiler changes: relevant generated C, assembly, semantic tests, and simulator or hardware results may also be required.

There is no build or installation procedure yet because the compiler implementation has not started. Early contributions focus on the pinned toolchain baseline, direct-register C hardware baseline, STM8 target contract, and MIR extraction.

## Vendor documentation

ST reference manuals are treated as external, versioned engineering dependencies. They are linked from [`docs/references/manifest.toml`](docs/references/manifest.toml) but are not distributed in this repository. Local copies belong in `docs/references/vendor/`, which is excluded from Git.

## License

Original project content is dual-licensed under either of:

- [Apache License, Version 2.0](LICENSE-APACHE)
- [MIT License](LICENSE-MIT)

at your option.

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in `stm8-rs` is dual-licensed as above, without additional terms or conditions.

Third-party components and documents retain their original licenses and are not relicensed by this project. See [THIRD_PARTY.md](THIRD_PARTY.md).
