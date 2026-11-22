# Ultimate Renamer

> **It Renames Everything.** A lightweight, native Linux batch-renamer with a dark-neon GUI — built in Rust, packaged as a single-file AppImage.

[![Build](https://img.shields.io/badge/build-passing-brightgreen)]()
[![Audit](https://img.shields.io/badge/cargo--audit-clean-brightgreen)]()
[![Platform](https://img.shields.io/badge/platform-Linux%20x86__64-blue)]()
[![License](https://img.shields.io/badge/license-MIT-lightgrey)]()

---

## Table of Contents

- [Why Another Renamer?](#why-another-renamer)
- [Feature Matrix](#feature-matrix)
- [Renaming Operations](#renaming-operations)
- [Files, Folders & Bulk Selection](#files-folders--bulk-selection)
- [The Live Preview](#the-live-preview)
- [UI / UX Design Notes](#ui--ux-design-notes)
- [Architecture](#architecture)
- [Safety & Correctness](#safety--correctness)
- [Build From Source](#build-from-source)
- [Packaging (AppImage)](#packaging-appimage)
- [Quality Gates](#quality-gates)
- [Roadmap / Non-Goals](#roadmap--non-goals)
- [Developer Contact](#developer-contact)

---

• Strategic Recommendation: Frames Lubuntu as the primary native development environment for guaranteed stability.
• Technical Breakdown: Bullet points highlighting the streamlined configuration, low resource footprint, and its ability to revive older hardware with zero lag.
• Veteran Insight: Concludes with an engaging nod to the fun of managing multiple Linux environments.

## Why Another Renamer?

Most bulk renamers are either heavyweight (Electron / Qt giants that pull half a desktop environment into your RAM) or anemic (a single text box with a fixed prefix). Ultimate Renamer is the middle ground the platform was missing:

- **Small native binary.** No JS runtime, no webview, no theme engine. Just Rust + FLTK. The release binary is a few megabytes; the AppImage ships in under 2 MB.
- **Low resource footprint.** Idles at a handful of megabytes of RSS. Runs on everything from a Raspberry Pi-class box to a Plasma workstation.
- **Every rename primitive in one place.** Prefix/suffix, regex find/replace, positional insert, substring removal, case transforms, numbering, extension surgery, truncation and whitespace cleanup — plus files *and* folders.
- **Deterministic, collision-safe execution.** A two-phase rename engine handles swaps and cycles correctly, and never clobbers an unrelated file.

The whole thing runs on X11 / XWayland, so it works across distros — Debian/Ubuntu, Fedora, Arch, openSUSE, ParrotSec, and Wayland sessions through XWayland.

---

## Feature Matrix

| Capability | Status | Notes |
|---|---|---|
| Batch file rename | ✅ | Multi-select via native file dialog |
| Batch folder rename | ✅ | Multi-select, folders treated as extensionless names |
| Add all files from a folder | ✅ | Non-recursive, skips dotfiles |
| Prefix / Suffix | ✅ | Applied to the name stem, extension preserved |
| Find & Replace | ✅ | Literal or regex, case-sensitive toggle |
| Insert at position | ✅ | From start or end, character-indexed |
| Remove text | ✅ | Literal or regex |
| Case conversion | ✅ | lower / UPPER / Title / Sentence / iNVERT |
| Sequential numbering | ✅ | Prefix, suffix, start, step, zero-pad, keep-original |
| Extension replace / add / remove | ✅ | Multi-dot aware (`archive.tar.gz`) |
| Truncate + whitespace cleanup | ✅ | trim / underscore / dash / collapse |
| Live preview with type column | ✅ | Recomputed on every keystroke |
| Working scrollbars (mouse + keyboard) | ✅ | Verified end-to-end |
| Borderless custom-drawn window | ✅ | Hand-rolled min / max / close glyphs |
| Maximize scales entire UI | ✅ | Work-area aware, stays in taskbar |
| AppImage with icon + desktop entry | ✅ | Single-file, portable |

---

## Renaming Operations

The left panel exposes eight mutually-exclusive modes, all operating on the selected items:

1. **Prefix / Suffix** — concatenate text before and/or after each stem.
2. **Find & Replace** — literal substring or full regex; optional case-insensitive matching.
3. **Insert At** — inject a string at a character index, counted from the beginning or the end.
4. **Remove** — delete a literal substring or regex match everywhere it appears.
5. **Case** — `lowercase`, `UPPERCASE`, `Title Case`, `Sentence case`, `iNVERT cASE`.
6. **Numbering** — configurable prefix/suffix, start value, step and zero-padding; optionally retains the original name after the number.
7. **Extension** — replace, append, or strip the extension. Multi-dot names are handled correctly.
8. **Advanced** — truncate to *N* characters (from either end) plus whitespace normalization.

Every input change triggers an immediate preview recompute — no "apply and hope" workflow.

---

## Files, Folders & Bulk Selection

Three acquisition paths feed the same queue:

- **ADD FILES** — native multi-file picker.
- **ADD FOLDERS** — native multi-directory picker; the *folder names themselves* are renamed.
- **ADD FOLDER FILES** — pick one folder, load every non-hidden file inside it.

Folders are modeled as extensionless entries so a directory named `my.project.v2` is treated as a single stem, not `my.project` + `v2` extension. Hidden dotfiles/dot-directories are skipped to avoid accidental damage. The preview marks each row with `F` (file) or `D` (directory) so the operator always knows what will change.

---

## The Live Preview

The right panel is the operator's source of truth:

- **Location line** — shows the parent directory of the queued items (or `(none)` when empty).
- **Three aligned columns** — `ORIGINAL`, `NEW NAME`, `STATUS` — with widths computed from the *live pixel width* of the widget. On a maximized 1920×1035 window, the columns stretch to fill the screen instead of clustering in the top-left.
- **Status per row** — `ready`, `auto-safe` (a conflicting name was auto-suffixed), or `unchanged`.
- **Scrolling** — no line wrapping; long filenames create real horizontal overflow. Both scrollbars are usable via mouse wheel, Shift+wheel, arrow keys, PageUp/PageDown, Home/End, and direct dragging.

---

## UI / UX Design Notes

This is not a stock FLTK look. The entire surface is drawn by hand:

- **Dark neon palette** — near-black backgrounds `(8,8,13)` / `(14,14,22)`, luminous green/cyan/red/yellow accents.
- **Metallic 3D buttons** — every control uses a shared gradient renderer: layered glow halos, a vertical metallic sheen, top highlight, bottom shadow and a neon border. The mode buttons (Prefix … Advanced) deliberately share the exact renderer as the action buttons so the UI reads as one coherent control family.
- **Window controls on the top-left** — close is a red glowing circle with an ×, maximize a green glowing square, minimize a yellow glowing minus. A CONTACT button sits top-right.
- **Title & subtitle** — `Ultimate Renamer` in red glow, `It Renames Everything` in green glow, centered at the top of the window. The green accent bar was removed after user feedback; a full-width divider line now closes the header cleanly at the right border.
- **Borderless but managed** — FLTK normally sets `_NET_WM_STATE_SKIP_TASKBAR` and `override_redirect` for borderless windows. We deliberately keep the window WM-managed and strip decorations via X11 `_MOTIF_WM_HINTS` after `show()`, so the app appears in the taskbar/tray while still having no native chrome.
- **Maximize** — resizes to the monitor's work area (excluding the panel) and proportionally re-lays-out every widget from recorded base rectangles, so the whole UI scales instead of leaving a dark empty region.

Uniform typography: all button labels render at font size 14; the preview uses Courier 14 for alignment and legibility.

---

## Architecture

```
.
├── Cargo.toml / Cargo.lock      # fltk, open, regex
├── src/
│   ├── main.rs                  # UI, drawing, state, event handling, X11 hints
│   ├── engine.rs                # pure rename logic + planning + apply
│   └── lib.rs                   # library surface for tests
└── tests/
    └── engine_test.rs           # 9 unit tests incl. real filesystem rename
```

**`engine.rs`** is deliberately UI-agnostic. It exposes:

- `build_name(original, index, mode, settings, is_dir)` — pure name transformation.
- `plan(files, mode, settings)` — produces `PlanItem`s, resolving collisions by appending ` (n)` while reserving names within the batch.
- `apply(plan)` — two-phase rename: everything moves to unique temp names first, then to final names, with rollback on failure. This is what makes swaps (`a→b`, `b→a`) work instead of failing or clobbering.
- `common_parts(...)` — longest common prefix/suffix and repeated-word detection helper.

**`main.rs`** owns an `AppState` behind `Rc<RefCell<..>>`, collects generic `LayoutItem` handles for proportional resize, and installs custom `draw`/`handle` callbacks on FLTK widgets. The X11 decoration stripping lives in a small `#[cfg(target_os = "linux")]` FFI module.

---

## Safety & Correctness

- `cargo audit` is clean — no known vulnerable dependencies.
- Collision handling never overwrites a file outside the batch; within a batch, final names are reserved before execution.
- Two-phase apply provides rollback semantics on partial failure.
- Dotfiles/dot-directories are excluded from folder scans.
- The contact URL exists in exactly one place in the source: the CONTACT button handler. It is not duplicated elsewhere in the UI or code.

---

## Build From Source

```bash
# Prerequisites: Rust toolchain, CMake, and X11 development headers
# (Debian/Ubuntu: sudo apt install cmake libx11-dev libxext-dev)
git clone <your-repo-url> ultimate_renamer
cd ultimate_renamer
cargo build --release
./target/release/ultimate_renamer
```

Run the test suite:

```bash
cargo test --release
```

Static analysis:

```bash
cargo install cargo-audit
cargo audit
```

---

## Packaging (AppImage)

The project ships a single-file AppImage built with `appimagetool`. The staging layout is conventional:

```
AppDir/
├── AppRun                       # exec wrapper exporting XDG paths
├── ultimate_renamer.desktop     # desktop entry
├── ultimate_renamer.png         # 256×256 neon "UR" icon
└── usr/
    ├── bin/ultimate_renamer
    └── share/
        ├── applications/ultimate_renamer.desktop
        ├── icons/hicolor/256x256/apps/ultimate_renamer.png
        └── metainfo/ultimate_renamer.metainfo.xml
```

Build command:

```bash
appimagetool AppDir Ultimate_Renamer-x86_64.AppImage
chmod +x Ultimate_Renamer-x86_64.AppImage
./Ultimate_Renamer-x86_64.AppImage
```

No installation required — download, `chmod +x`, run. Works on any glibc-based x86_64 distro with X11/XWayland.

---

## Quality Gates

| Gate | Status |
|---|---|
| `cargo build --release` | ✅ clean |
| `cargo test --release` | ✅ 9/9 passing (incl. real filesystem rename) |
| `cargo audit` | ✅ 0 vulnerabilities |
| Manual GUI verification | ✅ normal + maximized rendering, min/max/close, scrolling, taskbar visibility |
| Packaging | ✅ AppImage boots, correct WM hints, icon present |

---

## Roadmap / Non-Goals

The current release is considered feature-complete for its scope. Deliberate non-goals:

- Recursive folder-tree traversal in "ADD FOLDER FILES" (flat, top-level only).
- Undo/redo history and saved rename presets.
- Wayland-native backend (uses XWayland; sufficient across target distros).
- Drag-and-drop queue reordering.

These are not defects — they are scope boundaries. The core rename pipeline is stable and verified.

---

## Developer Contact

Click the **CONTACT** button in the application's top-right corner to open the developer's GitHub repositories in your default browser.

---

*Built with Rust, FLTK, and an unreasonable amount of attention to glow radii.*
