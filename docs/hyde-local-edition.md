# HYDE OS local edition

This fork is based on WordCraft 0.18.1, with HYDE's file-host bridge Save As correction and table-style indentation preservation. The application remains MIT OR Apache-2.0; retain the upstream copyright, NOTICE, font licences and contributor credits. It is a HYDE OS edition based on WordCraft by ArtCraft, not an official ArtCraft release. No asset from `docs/brand/` is included in the web distribution.

## Table indentation

The DOCX importer and exporter preserve `w:tblPr/w:tblInd` in table styles, including an explicit zero. Effective indentation resolves direct table formatting first, then the inherited table style, with a finite zero fallback. Layout, AutoFit, left-edge resize and the table-properties form use that same resolution. A child style does not acquire duplicated inherited XML on save.

The implementation adapts the MIT/Apache upstream fixes from [WordCraft PR 5386](https://github.com/storytold/wordcraft/pull/5386), head `87e6ee779b4e32accfe233c435bf242b8ad61818`, and [PR 5767](https://github.com/storytold/wordcraft/pull/5767), head `0c2c59f516026a4c053f4a608ace4a711a01bc47`, to this pinned release. The existing table-properties unit conversion and bounds remain in use.

Focused native regressions:

```sh
cargo test --locked -p wordcraft-docx table_style_indent
cargo test --locked -p wordcraft-layout table_style_indent_applies_without_a_table_indent
cargo test --locked -p wordcraft-engine style_indent
```

The HYDE integration repository's `Scripts/ArtCraft/Compare_Office.mjs --corpus --app wordcraft` opens its authored seven-page bilingual DOCX, edits it in the native web editor, saves, independently renders the result through the certified private Office converter, then opens and saves it in a fresh editor. `Verify_Office_Structure.py` checks content, headers, media, tables and RTL metadata. The comparison is representative acceptance, not a promise of universal Microsoft Office layout compatibility.

## Reproducible web inputs

- Source: the exact fork commit recorded in HYDE's `app/lib/artcraft/catalog.json`, with its checked-in `Cargo.lock`.
- Rust `1.100.0`, target `wasm32-unknown-unknown`.
- Trunk `0.21.14`; Apple arm64 archive SHA-256 `764e299dd50d89442a4e96a236349f57961984b701e74d3dbdb39cd1c9f5101e`.
- craft-fonts `b076417b5880bd4154bd22f00695b1d28c6bea83`; use its unchanged manifest and verify every included font against its recorded SHA-256. This release's web build selects the Arabic and Georgian faces, as upstream does. Preserve their OFL notices.
- craft-i18n `1e515422afdda2f317af430c1f9d68a5f83935b4`, also recorded in `craft-i18n.rev`; include WordCraft/common catalogues and `language-data` with their licences. Release builds must not silently become English-only.
- HYDE's shared bridge supplies its existing pinned Noto Sans Hebrew font through the editor's public font API before file opening. Reuse that bridge rather than adding a second font-delivery path.

Set absolute `CRAFT_FONTS_DIR` and `CRAFT_I18N_DIR`, `CRAFT_FONTS_REQUIRED=1`, `CRAFT_I18N_REQUIRED=1`, and `WORDCRAFT_BUILD_SHA` to the exact source commit. Use a task-owned target/cache and a bounded `CARGO_BUILD_JOBS` to share the self-hosted fleet. From `apps/wordcraft-web` run:

```sh
trunk build --release --locked --skip-version-check
```

Record the emitted archive's size and SHA-256, compiler/linker helper versions and font/catalogue input hashes in the HYDE runtime receipt. Verify the actual compiled web editor before updating the catalogue and admitted runtime package. Keep the previously admitted package available for rollback.
