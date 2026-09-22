# Meridian capstone — repository conventions

## Research wiki

A research wiki lives in `wiki/`. It is maintained by Claude, not by hand.

**Before creating, editing, or reading wiki pages, read `wiki/CLAUDE.md`.**
It defines the page formats, the ingest / query / lint workflows, and the
data-handling rule that governs what may enter the wiki.

Source documents live in `raw/` and are immutable — read them, never edit them.

## Data handling

No client data classified Restricted may enter this repository or any AI tool.
See `docs/superpowers/specs/2026-09-20-data-handling-checklist-design.md`.
