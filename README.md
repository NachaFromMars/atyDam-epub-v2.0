# atyDam-epub-v2.0 — Automated PDF→EPUB conversion (plain text)

> A 9-phase pipeline that turns messy PDFs into clean, validated EPUBs. Built for Buddhist texts, academic books, and multi-script documents — the plain variant with no bold emphasis.

[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-blueviolet)](https://github.com/NachaFromMars)

## Overview
atyDam-epub-v2.0 is an automated conversion pipeline that transforms PDF source files into polished EPUB e-books. It runs a smart 9-phase process with a closed-loop audit, handling tricky cases like broken Vietnamese and Pali font encoding. This is the base/plain version — it produces clean, readable EPUBs without added bold emphasis. For GPT-style bold keyword highlighting, use v3.0 instead.

## Phases
| Phase | Step |
|---|---|
| 1 | Extract raw text and structure from PDF |
| 2 | Clean broken encoding (Vietnamese, Pali) |
| 3 | Rebuild document structure and hierarchy |
| 4 | Extract and embed images |
| 5 | Reconstruct tables |
| 6 | Handle footnotes |
| 7 | Build EPUB package |
| 8 | Validate EPUB |
| 9 | Package final output |

## Usage / Quick Start
Trigger with a PDF conversion request. The pipeline runs all 9 phases automatically and delivers a validated EPUB.

## Trigger Keywords (OpenClaw)
convert pdf sang epub, đóng epub, pdf to epub, pdf2epub, chuyển pdf thành epub

## Related Skills
- [atyDam-epub-v3.0](https://github.com/NachaFromMars/atyDam-epub-v3.0) — same pipeline + GPT-style bold emphasis (Phase B10)

---
Part of the [NachaFromMars](https://github.com/NachaFromMars) OpenClaw skill ecosystem.
