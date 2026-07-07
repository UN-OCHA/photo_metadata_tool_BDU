# OCHA Photo Metadata Tool

A simple, **offline** web app to add OCHA-compliant IPTC/XMP metadata to photos
before uploading them to the MediaValet photo library — **no Adobe Lightroom
licence needed.**

**▶ Live tool:** https://un-ocha.github.io/photo_metadata_tool_BDU/

## Why

Partners and country offices must add standardized metadata (caption, credit,
copyright, keywords, location) and rename photos to the OCHA convention before
uploading to MediaValet. The official workflow uses Adobe Lightroom — but not
everyone has a licence, and some offices have weak internet. This tool fills that
gap.

## What it does

- Adds OCHA-standard IPTC/XMP metadata: creator, credit line, copyright, a
  structured *where · when · what · credit* caption, and keywords.
- UN-standard country dropdown with an auto-filled ISO code.
- Auto-builds the OCHA filename: `ISO_YYYY_MM_Org_Filename`.
- ReliefWeb keyword quick-pick and EXIF date pre-fill.
- Batch and single-photo modes.

## Offline &amp; private

The tool runs entirely in your browser. **Photos never leave your computer** —
nothing is uploaded anywhere. Once the page has loaded it works with no internet,
so it is usable in low-connectivity offices.

## Status

Prototype (Phase 0 — JPEG in / JPEG out). RAW input and further features are
planned. See [`SPEC.md`](SPEC.md).

## Design system

Colours and design tokens come from the shared **OCHA App Kit** (OCHA Common
Design System) and are kept in sync automatically — do not restyle the tool
directly. See [`CLAUDE.md`](CLAUDE.md).

## Project Owner

Javier Cueto, Head of the Brand and Design Unit (BDU), OCHA

## Maintained by

**OCHA Brand and Design Unit (BDU)**
- Team: ochavisual@un.org
- Focal point: Javier Cueto (cuetoj@un.org)
