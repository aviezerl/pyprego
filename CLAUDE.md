# pyprego Development Guide

## Branch Model

| Branch | Purpose | Remote |
|--------|---------|--------|
| `dev` | Active development. All work happens here. | `private` (aviezerl/pyprego) |
| `main` | Clean release history. One squash commit per release. | `origin` (tanaylab/pyprego) |

### Rules

- **All commits go to `dev` first.** Never commit directly to `main`.
- **`main` gets squash-merged releases only.** Each release is a single commit on `main` with a clean summary message.
- **Tags live on `main`.** Version tags (`v0.0.1`, `v0.0.2`, ...) are created on `main` after the squash commit.
- **Never add AI attribution to commits.** No `Co-authored-by` or similar AI-referencing trailers.

### Shipping (dev → main)

```bash
dev/skills/pyprego-ship/ship.sh                      # dry run
dev/skills/pyprego-ship/ship.sh "v0.0.1: summary"    # commit only  
dev/skills/pyprego-ship/ship.sh "v0.0.1: summary" --push  # full ship
```

### Dev-only paths (MUST NOT appear on `main`)

- `CLAUDE.md`
- `AGENTS.md`
- `DECISIONS.md`
- `PROGRESS.md`
- `dev/` (if created)
- `.a5c/`

## Testing

```bash
# Fast tests (~11 seconds, 614 tests)
pytest tests/ --ignore=tests/test_high_level.py --ignore=tests/test_integration.py --ignore=tests/test_ported_screening.py

# Full suite (includes slow regression-based tests)
pytest tests/
```

## Architecture

See `DECISIONS.md` for detailed architecture decisions.

- **NumPy-based** computation (no GPU/PyTorch dependency)
- **pandas DataFrames** for PSSMs and spatial models
- **Optional pymisha** for genomic integration
- **Clean array interfaces** for future GPU/differentiable version

## Key Performance Notes

- C extension (`_pyprego`) accelerates `init_energies`, `kmer_matrix`, and `choose_best_move`
- Performance matches or beats R/C++ across all operations
- Pure Python/NumPy fallback when C extension is not compiled
- Build C extension: `pip install -e .`

## Related Packages

- **prego** (R): The original R package this was ported from
- **pymisha**: Python misha genome database (optional dependency)
- **pyrego**: GPU-focused PyTorch implementation (separate project, different design)
- **iceqream**: Future Python port will depend on pyprego
