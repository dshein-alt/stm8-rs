# Vendor reference documents

This directory tracks third-party documentation used by `stm8-rs`. The referenced documents remain the property of their respective publishers and are not covered by any license granted for original project content. See [THIRD_PARTY.md](../../THIRD_PARTY.md).

Vendor PDFs are not committed to the repository. `manifest.toml` is the source of truth for document identities, official download URLs, revisions, publication dates, and local-copy hashes.

## Current references

| ID | Document | Revision | Publisher |
|---|---|---|---|
| [RM0016](https://www.st.com/resource/en/reference_manual/cd00190271.pdf) | STM8S Series and STM8AF Series 8-bit microcontrollers | Rev 14, October 2017 | STMicroelectronics |
| [PM0044](https://www.st.com/resource/en/programming_manual/cd00161709.pdf) | STM8 CPU programming manual | Rev 3, September 2011 | STMicroelectronics |

## Local reference cache

Download documents individually from the official URLs in the manifest and store them under:

```text
docs/references/vendor/
```

That directory is excluded from Git. Keep the original filenames recorded as `local_filename` in the manifest. Check the document ID, title, and revision before relying on a download.

After downloading a document, calculate its hash and record the lowercase hexadecimal result in the corresponding `sha256` field. For example:

```sh
sha256sum docs/references/vendor/cd00190271.pdf
```

An empty `sha256` field means that no local-copy hash has been recorded. A locally calculated hash identifies the exact downloaded bytes; it establishes authenticity only when it matches a checksum published by the vendor.

When a vendor updates a document, verify the new revision and publication date and update the manifest together with the local-copy hash. Do not describe a new download as a previously verified revision merely because it uses the same URL.

## Redistribution

`redistribution_permitted = false` means that the project has not identified permission to redistribute the document. It does not assert that no such permission could exist under a separate document-specific license or written authorization.

Do not add a vendor document to Git unless its redistribution terms have been verified and recorded. Public availability for download is not, by itself, redistribution permission. Use of ST materials is subject to the [STMicroelectronics Terms of Use](https://www.st.com/content/st_com/en/about/terms-of-use.html) and any terms accompanying the individual document.
