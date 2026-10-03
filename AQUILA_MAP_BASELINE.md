# Aquila Map Baseline (pre-migration, authoritative)

- Baseline commit: `f847d3caca6e23d4ad0cfed965698c2f54412e74` (2025-09-09, "Aquila powrot (#317)")
- Local annotated tag: `aquila-baseline-2025-09-09` (points at the commit above; **not pushed**)
- Inventory file: [AQUILA_MAP_BASELINE.tsv](AQUILA_MAP_BASELINE.tsv) (733 tracked `.dmm` files, 49,541,029 bytes total)
  - Columns: `path`, `git_blob`, `sha256`, `size_bytes`, `tracked`, `commit`
  - sha256 of the inventory file itself: `3ef1644e16a1f2054b4da249e7f5adc4ea35e246e651ead8731052b0e0083891`
- 0 untracked `.dmm` files existed at baseline. All 733 are under `_maps/`.

**Authoritative column: `git_blob`.** The `sha256` column was taken from the Windows working tree (`core.autocrlf=true`), so it can differ from the committed bytes on another checkout. Use `git_blob` for verification; use `sha256` only on this machine.

The inventory is deliberately independent of Git history: after the migration begins, it is compared with the tree rather than with a ref.

## Verification procedure (run after every future migration stage)

```bash
# every baseline map must still have the same blob in the candidate tree
tail -n +2 AQUILA_MAP_BASELINE.tsv | while IFS=$'\t' read path blob rest; do
  now=$(git rev-parse "HEAD:$path" 2>/dev/null || echo MISSING)
  [ "$now" = "$blob" ] || echo "CHANGED: $path ($blob -> $now)"
done
# no new .dmm files unless explicitly approved
git ls-files '*.dmm' | sort > /tmp/now.txt
tail -n +2 AQUILA_MAP_BASELINE.tsv | cut -f1 | sort > /tmp/base.txt
comm -13 /tmp/base.txt /tmp/now.txt   # new maps (must be empty unless approved)
```

Every `CHANGED` line must map to an entry in the compatibility-modification log (see the migration analysis, "Map compatibility log"). Anything else is a migration failure until investigated.

## Known defects already present in the baseline (not fixed)

Three baseline maps contain **unresolved Git merge-conflict markers** committed by #317 (`<<<<<<< HEAD` … `=======` … `>>>>>>> d5afbe889c`; the second side is upstream `d5afbe889c`, 2025-05-01):

| Map | Conflict hunks | In `config/maps.txt`? | Effect |
|---|---|---|---|
| `_maps/map_files/Serendipity/Serendipity2.dmm` | 12 | `serendipity` – votable | Map parser error; not loadable |
| `_maps/map_files/Snake/snake_upper.dmm` | 1 | `snake` – votable | Not loadable |
| `_maps/map_files/Tycoon/Tycoon2.dmm` | 2 | `tycoon` (votable commented out) | DM compile fails with `-DALL_MAPS` (`Tycoon2.dmm:46720: unexpected input: <<`) |

Their baseline blobs are recorded in the TSV as they are. Which side counts as "authoritative Aquila" for these three files needs your decision before they are touched (see AQUILA_BASELINE_VALIDATION.md).
