# Documentation policy

This document defines how `stm8-rs` organizes and maintains project documentation.

## Organization

Keep the root `README.md` focused on the project overview, current status, and navigation. Place substantial aspect-specific material in separate, focused documents under `docs/` rather than expanding the README into a comprehensive manual.

Use stable, descriptive, unversioned filenames for living documents. Examples include:

- `docs/PROJECT_MANAGEMENT.md`
- `docs/DOCUMENTATION.md`
- `docs/references/README.md`
- `docs/references/manifest.toml`

Link between documents using relative Markdown links. Links should target the canonical document, not a copy or an outdated filename. Do not use local filesystem paths in repository documentation.

## Revision history

Living documents evolve in place through Git history. Do not create versioned copies for routine revisions, such as changing `PROJECT_MANAGEMENT.md` into `PROJECT_MANAGEMENT_v1.1.md`.

Document metadata may record a version or status when useful, but the canonical filename remains stable. Contributors should use Git history to inspect earlier revisions.

Versioned filenames are allowed only when the document is explicitly a versioned specification or historical artifact. The current exceptions are:

- `docs/roadmap/Rust_for_STM8_SDCC_Feasibility_and_Roadmap_v1.0.md` and its generated PDF;
- historical roadmap files under `docs/roadmap/_history/`.

New exceptions should be deliberate and apparent from the document's purpose and location.

## Source and generated formats

Use Markdown as the canonical format for project-owned living documentation because it supports readable reviews and diffs. Generated formats such as PDF may be included for distribution, but they do not replace the canonical Markdown source.

When a document with generated counterparts changes, regenerate those counterparts in the same change or explicitly record why they were not updated.

## Maintenance

When adding, removing, renaming, or restructuring a document:

1. Preserve a single canonical location for its current content.
2. Update all internal references, including README navigation and YAML frontmatter paths.
3. Keep related documentation consistent with the changed behavior or policy.
4. Check that all relative Markdown links resolve.

Vendor documentation follows the separate external-reference policy in [`references/README.md`](references/README.md); it is not copied into the project documentation hierarchy.
