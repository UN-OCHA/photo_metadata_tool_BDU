# OCHA Photo Metadata Tool — instructions for Claude

A self-contained, offline web app (`app/index.html`) that embeds OCHA-compliant
IPTC/XMP metadata into photos for upload to MediaValet. See `SPEC.md` for scope.

## Styling / colours come from the OCHA App Kit — do NOT restyle here

This tool consumes design tokens from the **OCHA App Kit**, the single source of
truth shared with other OCHA apps (e.g. QuickVid):

`…/OCHA_design_system/ocha-common-design-system-BDU/app-kit/ocha-app-kit.css`

- The tool's `:root` tokens live **between markers** in `app/index.html`:
  `/* OCHA-KIT:START */ … /* OCHA-KIT:END */`. That block is **generated** — never
  hand-edit it; `sync.py` overwrites it from the kit.
- This tool uses the kit's own variable names (`--ocha-cyan`, `--ocha-blue`,
  `--ink`, `--line`, `--input-border`, `--bg`, `--card`, `--ok`, `--warn`, `--radius*`
  …), so it inherits kit values directly — **no alias layer**.
- **To change a colour or token:** edit `ocha-app-kit.css`, then:
  ```bash
  cd "…/ocha-common-design-system-BDU/app-kit"
  python3 sync.py --app "Photo Metadata"
  ```
  Never change a hex or token value inside this tool. If a shared component needs to
  change, change it **in the kit** and sync.

Registered in the kit as an `inline` / `tokens`-scope app in `app-kit/apps.json`.

## Out of scope for the token connection (separate, bigger job)

The tool still uses its own component classes (`.card`, `button.primary`, etc.)
rather than the kit's `cd-*` classes. Migrating those to `cd-*` is a larger,
separate task and is **not** required for the token sync to work.

## Everything else

App logic, IPTC/XMP writing, country list, filename convention (`ISO_YYYY_MM_Org_File`),
keyword chips, EXIF prefill — all documented in `SPEC.md` and the project memory.
Layout-only CSS may live here; shared look-and-feel does not.
