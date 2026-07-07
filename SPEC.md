# OCHA Photo Metadata Tool — Specification

**Status:** Draft for review
**Author:** Prepared with Claude for the OCHA Brand and Design Unit
**Date:** 6 July 2026

---

## 1. Purpose

A simple, offline-capable web app that lets OCHA colleagues and partners add
OCHA-compliant metadata to photos **before** uploading them to the MediaValet
DAM — without needing an Adobe/Lightroom licence.

It replaces the current fallback in the partner guidance ("if you don't have
Adobe, use ExifTool") with a guided form that **encodes the OCHA rules directly**,
so non-technical field staff cannot get the format wrong.

### Problems it solves
- **No Adobe licence** — many country offices and partners lack Lightroom.
- **ExifTool is unusable for non-technical staff** — it's a command-line tool.
- **Weak/no internet** — country offices can't rely on cloud tools.
- **Inconsistent metadata** — free-text entry produces non-compliant captions,
  credit lines and filenames that break MediaValet's auto-description pull.

### Non-goals (v1)
- Not a photo *editor* (no cropping, colour, retouching).
- Not an uploader to MediaValet (the officer still uploads via MediaValet's
  normal flow — this tool only prepares the files).
- Not a selection/culling tool (partners still choose their best 6+ photos).

---

## 2. Users

| User | Context | Needs |
|---|---|---|
| Partner comms staff | Field, often weak internet, no Adobe | Dead-simple guided form, works offline |
| OCHA PIOs / HFU focal points | Uploading partner photos to MediaValet | Fast batch tagging, compliance validation |

Design principle (per BDU standard): **simple and visual**. A first-time user
with no training should produce a compliant file.

---

## 3. Architecture — why offline is inherent, not bolted on

The critical property: **the app needs no server.** Reading a photo and writing
metadata back into it both happen in the browser, on the user's own machine, in
JavaScript. No photo is ever uploaded during editing.

```
┌─────────────────────────────────────────────┐
│  Browser (Chrome/Edge) — runs fully offline  │
│                                              │
│  1. User picks a photo / folder (local)      │
│  2. App reads existing metadata (JS)         │
│  3. User fills OCHA form (batch + per-photo)  │
│  4. App writes IPTC/XMP into the file (JS)    │
│  5. User saves tagged files back to disk      │
└─────────────────────────────────────────────┘
        ↓ (later, whenever bandwidth allows)
   Officer uploads tagged files to MediaValet
```

Because nothing is uploaded during editing, **weak internet is irrelevant to the
metadata step.** Built as an installable **PWA** (Progressive Web App), the user
loads it once over any connection, then it runs with wifi completely off — even
relaunched days later.

### Distribution / hosting options (not mutually exclusive)
1. **Hosted PWA** on a UN/OCHA domain or GitHub Pages — installable, auto-updates
   when online, works offline after first load. *Recommended primary.*
2. **Single self-contained `index.html`** — emailed or on a USB stick, opened by
   double-click, no install, no server. Ultimate fallback for the hardest
   connectivity cases.

Both ship from the same codebase.

---

## 4. Metadata fields (mapped to OCHA partner guidance)

Straight from *Guidelines on Uploading Photos on MediaValet for Partners* (16 Apr 2026).
All fields written into the **IPTC/XMP** block that MediaValet reads on ingest.

| OCHA field | Written to (IPTC/XMP tag) | App behaviour |
|---|---|---|
| **Creator** | `dc:creator` / IPTC By-line | Text. Hint: use org name if photographer undisclosed. |
| **Credit Line** | `photoshop:Credit` | Auto-assembled `Org/Photographer` (e.g. `UNOCHA/Francis Mweze`). |
| **Copyright** | `dc:rights` / IPTC Copyright | Org name (UNOCHA or partner). |
| **Description (Caption)** | `dc:description` / IPTC Caption-Abstract | **Structured builder** — see §5. |
| **Keywords** | `dc:subject` / IPTC Keywords | Tag input + one-click **"US Grant"** toggle + a **quick-add chip palette** (tap to add/remove) built from the **ReliefWeb** controlled vocabulary — 20 themes + 20 disaster types. Aligns photo keywords with how OCHA/ReliefWeb classify content. |
| **Country/Region** | IPTC `Country/Region` | Dedicated field — the Lightroom guide fills this explicitly. |
| **ISO Country Code** | IPTC `ISO Country Code` | Dedicated field — used in both the metadata and the filename. |
| **City / State-Province** | IPTC `City` / `State/Province` | Optional dedicated location fields; improve MediaValet search. |

> **Confirmed from the internal Guidebook:** MediaValet's own required upload
> attributes are **Copyright, Country, Photographer** — re-entered in the MediaValet
> UI as a double-check, but auto-satisfied when embedded correctly. The tool should
> make these three bulletproof. Filling the dedicated Country/ISO fields (not just
> burying location in the caption) matches current OCHA practice.

### Filename (not metadata, but enforced)
**Format used by the tool:** `ISOCountryCode_YYYY_MM_OrganizationName_FileName`
(e.g. `LBN_2026_03_UNOCHA_DSC00349`). The app provides a **live validator + batch renamer**.

> **⚠️ Convention change (decided by Javi, BDU Head, 6 Jul 2026):** the existing OCHA
> guidance uses **`MMYYYY`** (month-first, e.g. `032026`), which sorts by month before
> year — chronologically wrong when browsing. The tool uses **`YYYY_MM`** (year-first,
> with the underscore) instead, matching OCHA's own MediaValet **folder** convention
> exactly (`YYYY_MM_ISO_theme`, e.g. `2026_03_BGD_...`) so staff see the same date
> format on folders and files. *(Note: an earlier draft used `YYYYMM` without the
> underscore for cleaner field-splitting, but Javi opted to match the folder format —
> the split-fields concern was moot anyway since original camera names already contain
> underscores.)* **Action:** the guidance PDFs and the partners' instructions should
> be updated to `YYYY_MM` so the whole system moves together.

---

## 5. Structured caption builder

The guidance requires the caption to contain **Where · When · What/Who · Credit**.
Instead of a free-text box, the app collects these as separate inputs and
auto-composes the compliant string:

```
Inputs:
  Where (City/region, Country):  Beirut, Lebanon
  When (Month, Year):            March 2026
  What/Who:                      A distribution of emergency food assistance to
                                 displaced families affected by the conflict.
  Credit (Org/Photographer):     Medair/John Doe

→ Composed caption:
  "Beirut, Lebanon, March 2026 – A distribution of emergency food assistance to
   displaced families affected by the conflict. Photo: Medair/John Doe"
```

The composed caption is editable before saving. Optional prompts for full names
and original quotes, per guidance.

---

## 6. Batch vs individual editing

- **Batch (shared fields):** Creator, Credit, Copyright, Org, Country, Month,
  Keywords, US Grant toggle → applied across the whole selected set in one action.
- **Per-photo (unique fields):** the "What/Who" of the caption and the filename
  differ per shot → the app steps through each photo with a thumbnail preview so
  the officer completes only what's unique.

This mirrors the Lightroom batch-then-refine workflow the guidance describes.

### Scaling to many photos (agreed future design — not yet built)

The Phase 0 prototype uses a simple vertical list, which is fine for a handful of
photos but unwieldy at 60+. The agreed model for large sets is a **two-tier grid +
selection** UI (confirmed 6 Jul 2026, deferred until after RAW / MediaValet test):

- **Tier 1 — "Applies to all N photos"** panel: the shared general fields
  (photographer, org, copyright, country, date, keywords). Set once.
- **Tier 2 — thumbnail grid**: multi-select photos, then write a **description for
  just that selection**. Handles the real granularity — one photo, 2–3 sharing a
  caption, or many groups — because a "group" is simply "the photos you selected"
  (no new concept). Individual edit = select one.
- **Key affordances:** a **"select photos with no description yet (N)"** helper for
  the knock-out workflow so nothing is missed; per-tile indicators (✓ has a
  description, colour dot = photos sharing one caption); `mixed` shown when a field
  differs across the selection (edit overwrites only the selected).
- Chosen over the pure Lightroom single-panel model to reduce accidental
  overwrite-all for non-technical users, while still matching the familiar
  select-then-edit interaction from their Lightroom training.

---

## 7. File format handling — RESOLVED from the OCHA guidance

**The MediaValet workflow is JPEG-only by design.** The internal Guidebook
(*Editing Metadata and Uploading to MediaValet*, June 2025) makes this explicit:

- **Step 6 — Final Check and Export:** photos are exported from Lightroom as
  **Image Format: JPEG, Quality 100, sRGB** *before* upload.
- Every asset in both upload guides is a `.jpg` / `.JPG`.
- **Step 7** warns: *"Do NOT add a description — it will override the embedded
  metadata"* → MediaValet reads metadata **embedded in the JPEG**.

So RAW never reaches MediaValet: the current workflow uses Lightroom to convert
RAW → JPEG and embed metadata in one pass. **There is no XMP-sidecar question** —
MediaValet only ever ingests JPEGs, and there's nothing to sidecar. *(This was the
earlier "critical dependency"; the docs resolve it — no MediaValet test needed.)*

### Inputs and output

**Output is always a compliant JPEG** (the MediaValet deliverable), with OCHA
metadata embedded. Accepted **inputs**:

| Input | Handling | Notes |
|---|---|---|
| **JPEG** | Embed IPTC + XMP directly (APP1/APP13), re-save. ✅ Well-supported in JS. | The common case (phone/camera JPEGs). |
| **RAW / DNG** (CR2/CR3, NEF, ARW, RAF…) | **Extract the camera's embedded full-size JPEG preview**, embed metadata into it, output as JPEG. WASM RAW-decoder fallback for files without a usable preview. | Serves the "shoots RAW, no Lightroom" user. See below. |
| **PNG / TIFF** | Embed XMP directly. Nice-to-have. | Not the upload format; low priority. |

### Why the embedded-preview approach for RAW

The tool exists for people **without Lightroom**, but some of them shoot RAW. Rather
than decode RAW pixel-by-pixel (heavy, and a naive decode renders flat without camera
profiles), the tool extracts the **full-size JPEG preview the camera already embeds
inside every RAW** — i.e. the camera's own rendering, which looks correct. Metadata
is written into that JPEG. This is lightweight, offline, and format-agnostic. A WASM
RAW decoder is the fallback only when no adequate preview exists.

### 🚫 Explicitly out of scope: image editing

**No image adjustments of any kind** (decided). The tool is metadata-only:
- No RAW development (exposure, white balance, highlights/shadows) — that's a
  browser re-implementation of Lightroom's Develop module; anyone needing it already
  uses Lightroom. Out of scope by design.
- No JPEG tweaks either (brightness/contrast/crop/straighten) in v1 — keeps the tool
  simple and focused. Can be revisited later if users ask, but not planned.
- *(Note: aperture and other capture settings cannot be changed post-shot regardless.)*

---

## 8. Compliance & safety features

- **Pre-upload checklist** (from guidance) as tick-boxes before save: suitable for
  public comms · filename valid · caption compliant · no sensitive content ·
  privacy protected (minors/GBV) · dignity/ethical visuals · partner authorisation ·
  informed consent obtained.
- **Filename validator** — flags anything not matching `ISO_YYYY_MM_Org_File`.
- **Required-field guard** — won't export until Creator, Credit, Copyright and
  Caption are present.
- These are *aids*, not enforcement — consent/dignity remain human judgment calls.

---

## 9. Technology

Deliberately minimal for long-term BDU maintainability:

- **Static site, no backend.** Vanilla JS (or a very light framework) — no build
  server, no database, no API keys.
- **Read metadata:** `exifr` (reads EXIF/IPTC/XMP across JPEG, TIFF, PNG, HEIC and
  many RAW formats).
- **Write metadata:** JPEG via IPTC/XMP segment writing (`piexifjs` for EXIF +
  XMP packet injection); XMP for PNG/TIFF/DNG; plain-text `.xmp` sidecars for RAW.
- **Local files:** File System Access API for in-place folder read/write
  (Chrome/Edge). Fallback for Firefox/Safari: download tagged copies (+ sidecars)
  as a zip via `JSZip`.
- **Offline:** service worker + web app manifest → installable, fully offline.
- **Branding:** OCHA visual identity (UN Blue, Roboto) via the BDU design tokens.
- **i18n-ready:** all UI strings externalized into a single strings file from day one.
  Ships **English-only** in v1, but adding FR/ES/AR (incl. RTL for Arabic) later is a
  translation task, not a rewrite.

### Browser support
- **Chrome / Edge (Chromium):** full experience incl. save-in-place. *Primary.*
- **Firefox / Safari:** works via download-copies fallback.

---

## 10. Phased delivery

| Phase | Scope | Output |
|---|---|---|
| **0 — Prototype** ✅ | JPEG in/out, single + batch, structured caption, filename tool, download output | **Built + verified** → `app/index.html` |
| **1a — Smart assists** ✅ | EXIF date → prefill month/year (no AI); ReliefWeb keyword quick-pick chips | Built + verified |
| **1 — RAW input** | Extract embedded JPEG preview from RAW/DNG → tag → JPEG out; WASM decoder fallback | Handles the RAW-shooter case |
| **2 — Compliance** | Checklist, validators, required-field guard, OCHA branding | Reviewable app |
| **3 — Offline PWA** | Service worker, manifest, install, File System Access save-in-place | Installable offline app |
| **4 — Polish/deploy** | Hosting, docs, short how-to video | Production tool |

---

## 11. Open questions

1. ~~**MediaValet + XMP sidecars**~~ — ✅ *Resolved from the docs: MediaValet workflow
   is JPEG-only; RAW is exported to JPEG in Lightroom before upload, so there is no
   sidecar to worry about and no test needed.*
2. ~~**RAW input**~~ — ✅ *Resolved: accept RAW/DNG via embedded-preview extraction →
   JPEG out (§7). Metadata-only; no image editing.*
3. ~~**Languages**~~ — ✅ *English for v1, built i18n-ready (§9) for later FR/ES/AR.*
4. ~~**Location fields**~~ — ✅ *Resolved: use dedicated IPTC Country/Region + ISO
   Country Code fields (matches current OCHA Lightroom practice), plus optional
   City/State.*
5. **Hosting home** — GitHub Pages under `{project}_BDU`, a unocha.org page, or both?

---

## Project Owner
Javier Cueto, Head of the Brand and Design Unit (BDU), OCHA

## Maintained by
**OCHA Brand and Design Unit (BDU)**
- Team: ochavisual@un.org
- Focal point: Javier Cueto (cuetoj@un.org)
