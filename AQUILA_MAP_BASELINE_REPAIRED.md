# Aquila Map Baseline: REPAIRED (candidate, pending approval)

- Derived from original baseline `f847d3caca` (tag `aquila-baseline-2025-09-09`) plus the documented Phase 0.5 repairs in `AQUILA_PHASE_0_5_BASELINE_REPAIR.md`.
- Inventory: [AQUILA_MAP_BASELINE_REPAIRED.tsv](AQUILA_MAP_BASELINE_REPAIRED.tsv), 733 `.dmm` files. sha256 of the inventory file: `225605d18a88014805d12dea08de12954700b27fee67b81084a6cad53bdfa88c`.
- `git_blob` is the blob committed in `34fa11431f` ("Fix pre-migration Aquila baseline issues").
- The original `AQUILA_MAP_BASELINE.*` is untouched and still describes the broken `f847d3caca` state.

## Maps that differ from the original baseline (8 of 733)

| Map | Change |
|---|---|
| `Tycoon/Tycoon2.dmm` | conflict markers removed (2 hunks) |
| `Snake/snake_upper.dmm` | conflict markers removed (1 hunk) |
| `Serendipity/Serendipity2.dmm` | conflict markers removed (12 hunks) |
| `Serendipity/Serendipity1.dmm` | silent merge corruption repaired (2 undefined keys, 42 tiles) |
| `Atlas/atlas.dmm`, `Aquila_Atlas/Aquila_Atlas.dmm`, `Atlantis/Atlantis.dmm`, `Aquila_Snake/Aquila_Snake_Lower.dmm` | compatibility log C1: `deck_turret` → `deck_turret/core` |

The other 725 maps are byte-identical to the original baseline. Verification script: same as in `AQUILA_MAP_BASELINE.md`, with this TSV.

After approval, this TSV becomes the migration's authoritative baseline: every future stage must keep all 733 blobs equal to it, unless a new entry is added to the compatibility log.
