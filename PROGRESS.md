# pyprego Progress

## Completed

### Project scaffolding (2026-04-03)
- [x] pyproject.toml with full metadata, dependencies, tool config
- [x] Package directory structure: pyprego/ with all modules
- [x] tests/ directory with conftest.py and fixtures
- [x] py.typed marker for PEP 561

### Core types (types.py)
- [x] `pssm_dataframe()` -- create PSSM DataFrame from (L,4) array
- [x] `pssm_to_array()` -- extract (L,4) array from PSSM DataFrame
- [x] `spatial_dataframe()` -- create spatial model DataFrame
- [x] `RegressionResult` dataclass with predict method and to_dict serialisation

### Utilities (utils.py)
- [x] `rc()` -- reverse complement of a DNA string
- [x] `rc_array()` -- reverse complement of a list of sequences
- [x] `validate_sequences()` -- validation and normalisation of input sequences
- [x] `calc_sequences_dinucs()` -- dinucleotide counts per sequence (matrix output)
- [x] `calc_sequences_dinuc_dist()` -- positional dinucleotide frequency distribution
- [x] `calc_sequences_trinuc_dist()` -- positional trinucleotide frequency distribution
- [x] `sample_quantile_matched_rows()` -- stratified sampling preserving quantile distribution

### PSSM operations (pssm.py) -- fully implemented (2026-04-03)
- [x] `consensus_from_pssm()` -- consensus sequence with IUPAC ambiguity codes
- [x] `pssm_rc()` -- reverse complement a PSSM (row reversal + A<->T, C<->G swap)
- [x] `bits_per_pos()` -- information content per position (R-compatible: log2(4) + sum(p*log2(p)))
- [x] `pssm_trim()` / `trim_pssm()` -- trim low-information edges
- [x] `pssm_add_prior()` -- add uniform prior and re-normalise
- [x] `pssm_cor()` -- Spearman/Pearson correlation between two PSSMs at best alignment (sliding window)
- [x] `pssm_diff()` -- symmetric KL divergence between two PSSMs at best alignment
- [x] `pssm_match()` -- match PSSM against motif database (dict or DataFrame), supports spearman/pearson/kl
- [x] `pssm_concat()` / `concat_pssm()` -- concatenate PSSMs with optional gap
- [x] `pssm_theoretical_max()` / `pssm_theoretical_min()` -- score bounds (R-compatible: log(reg + rowMax/Min))
- [x] `pssm_quantile()` -- linear interpolation between theoretical min/max
- [x] `pssm_to_kmer()` -- convert PSSM to consensus k-mer (rolling bits window, threshold)
- [x] `pssm_dataset_cor()` -- pairwise correlation matrix for a collection of PSSMs
- [x] `pssm_dataset_diff()` -- pairwise KL divergence matrix for a collection of PSSMs

### K-mer operations (kmers.py) -- fully implemented (2026-04-03)
- [x] `generate_kmers(k, alphabet, max_gap, min_gap)` -- generate all k-mers with optional gap/wildcard support
- [x] `kmer_matrix(sequences, kmers, max_gap)` -- count k-mer occurrences per sequence (supports gapped k-mers via regex)
- [x] `kmers_to_pssm(kmer, prior)` -- convert k-mer string(s) to PSSM DataFrame (R-compatible output with kmer/pos columns)
- [x] `pssm_to_kmer(pssm, kmer_length, pos_bits_thresh, prior)` -- convert PSSM back to k-mer (best window selection, bit threshold)
- [x] `screen_kmers(sequences, response, kmer_len, kmers, max_gap, min_gap, seed, min_cor)` -- screen k-mers for correlation with 1D/2D response
- [x] Unit tests: 30 tests in tests/test_kmers.py (all passing)

### PWM computation engine (compute.py) -- fully implemented (2026-04-03)
- [x] `compute_pwm()` -- PWM energy scoring with logSumExp/max aggregation, bidirectional, spatial weighting, prior
- [x] `compute_local_pwm()` -- per-position PWM scoring with bidirectional support and spatial weighting
- [x] Internal helpers: `_encode_sequences`, `_prepare_pssm`, `_prepare_pssm_local`, `_compute_log_pssm`, `_score_windows`, `_log_sum_exp`
- [x] Faithful port of C++ `DnaPSSM::integrate_energy` / `integrate_energy_max` scoring logic
- [x] N-base handling (average log-prob, matching C++ behavior)
- [x] Reverse complement scoring (reversed PSSM + complemented bases)
- [x] Spatial binning with configurable bin size and per-bin factors
- [x] Unit tests: 51 tests in tests/test_compute.py (all passing), covering:
  - Basic scoring, perfect match vs mismatch ranking
  - logSumExp vs max aggregation
  - Bidirectional vs unidirectional
  - Spatial factors (uniform, high/low, multi-bin)
  - N-base handling (partial N, all-N)
  - Prior effect on score extremity
  - Consistency between compute_pwm and compute_local_pwm (max and logSumExp)
  - R compatibility tests using exact PSSM and sequence from test-compute_pwm.R
  - Edge cases (short sequences, lowercase, spat_min/spat_max, many sequences)

### Regression optimizer (regression.py) -- fully implemented (2026-04-03)
- [x] `regress_pwm()` -- main public API: motif discovery via coordinate descent PWM regression
- [x] `_PWMLRegression` engine class -- faithful port of C++ `PWMLRegression` class:
  - `init_seed()` -- initialise PSSM from k-mer string (* = wildcard)
  - `init_pssm()` -- initialise from pre-computed PSSM (K, 4) array
  - `add_responses()` -- set response data with per-fold statistics
  - `init_energies()` -- compute derivatives (product-of-probs over sliding windows, forward + RC)
  - `choose_best_move()` -- coordinate descent: evaluate 20 moves x K positions, rank across folds
  - `apply_move()` -- apply perturbation, clamp, renormalise
  - `optimize_spatial_factors()` -- greedy spatial bin optimisation (+/- step)
  - `optimize()` -- multi-phase loop with resolution decay until convergence
  - `compute_cur_r2()` / `compute_cur_r2_fold()` -- R-squared scoring
  - `compute_cur_ks()` / `compute_cur_ks_fold()` -- one-sided KS statistic scoring
  - `_compute_cur_r2_spat()` / `_compute_cur_ks_spat()` -- spatial score computation
  - Symmetrize spatial factors for bidirectional models
  - Cross-validation with configurable number of folds (rank-averaging)
- [x] Spatial helper functions: `_calculate_bins()`, `_calc_spat_min_max()`
- [x] Neighbourhood builder: `_build_neighbourhood()` -- 20 perturbation moves
- [x] PSSM DataFrame initialisation, spatial model initialisation, pre-computed model support
- [x] `predict()` function on RegressionResult using `compute_pwm()`
- [x] Full parameter parity with R/C++ (resolutions, spat_resolutions, log_energy, energy_epsilon, etc.)
- [x] Unit tests: 30 tests in tests/test_regression.py (all passing), covering:
  - Helper functions (encoding, neighbourhood, binning)
  - Optimizer improves score from initial k-mer (R2 and KS)
  - Binary regression with planted motif (KS metric)
  - Continuous regression with planted motif (R2 metric)
  - Multi-dimensional response
  - Planted motif discovery (consensus verification)
  - Bidirectional vs unidirectional
  - Spatial factor optimisation (spat-only, pwm-only modes)
  - predict() consistency with stored pred
  - PSSM DataFrame initialisation
  - Pre-computed spatial model initialisation
  - Result structure and serialisation
  - Input validation (metric, lengths, binary check)
  - Reproducibility with seed
  - Internal engine (init_energies, symmetrize_spat)
  - Cross-validation with multiple folds

### API surface defined (placeholders)
- [x] `motif_enrichment()` -- database screening (API defined, NotImplementedError)
- [x] `gextract_pwm()` -- genomic PWM extraction (API defined, NotImplementedError)

### Motif database (motif_db.py)
- [x] `create_motif_db()` -- create a motif database dict
- [x] `get_motif_pssm()` -- retrieve PSSM by name
- [x] `motif_db_to_dataframe()` -- flatten to long-form DataFrame

### Visualization (visualization.py) -- fully implemented (2026-04-03)
- [x] `plot_pssm_logo()` -- sequence logo via logomaker (falls back to stacked bar chart if logomaker unavailable)
- [x] `plot_spat_model()` -- spatial model line-and-point chart (matches R geom_line + geom_point)
- [x] `plot_regression_prediction()` -- predicted vs observed scatter with R-squared/r annotation
- [x] `plot_regression_prediction_binary()` -- 1-ECDF plot with KS D statistic annotation (matches R implementation)
- [x] `plot_regression_qc()` -- multi-panel QC figure (PSSM logo + spatial + prediction), auto-detects binary/continuous
- [x] `plot_regression_qc_multi()` -- multi-motif QC (per-motif logo/spatial/prediction rows + score summary panel)
- [x] All matplotlib imports are lazy (pyprego works without matplotlib installed)
- [x] tests/test_visualization.py -- 36 smoke tests (all passing)

### Genomic integration (genomic.py) -- fully implemented (2026-04-03)
- [x] `intervals_to_seq(intervals, size)` -- extract sequences from genomic intervals via pymisha, with optional size normalization
- [x] `gextract_pwm(intervals, pssm, ...)` -- extract PWM scores for genomic intervals (intervals -> sequences -> compute_pwm)
- [x] `gextract_pwm_quantile(intervals, pssm, quantiles, ...)` -- map PWM scores to quantiles from background distribution
- [x] `gextract_local_pwm(intervals, pssm, ...)` -- per-position PWM scores for genomic intervals (via compute_local_pwm)
- [x] `gintervals_center_by_pssm(intervals, pssm, size, ...)` -- re-center intervals on max PSSM score position
- [x] `_normalize_intervals()` -- internal helper for centering intervals to a target size
- [x] All pymisha imports are lazy (pyprego works without pymisha installed)
- [x] Mirrors R prego functions: intervals_to_seq, gextract_pwm, gextract_pwm.quantile, gextract.local_pwm, gintervals.center_by_pssm
- [x] Unit tests: 31 tests in tests/test_genomic.py (all passing), covering:
  - _normalize_intervals centering, column preservation, immutability
  - _require_pymisha error handling
  - intervals_to_seq: basic extraction, uppercase, size parameter, no-size, pymisha unavailable
  - gextract_pwm: scores array, match vs compute_pwm, bidirect, func=max, size forwarding
  - gextract_local_pwm: 2D shape, match vs compute_local_pwm, trailing NaN positions
  - gextract_pwm_quantile: 0-1 range, background intervals
  - gintervals_center_by_pssm: centering, extra columns, column order
  - Integration pipeline: planted motif scoring, local max at planted position, center shift
  - Error handling: all 5 functions raise ImportError without pymisha

### Test suite (2026-04-03)
- [x] tests/test_pssm.py -- 48 tests covering all PSSM operations (80 total with utils)
- [x] tests/test_utils.py -- 32 tests covering rc, dinucs, trinucs, sample_quantile_matched_rows
- [x] tests/test_kmers.py -- 30 tests covering k-mer operations (from previous session)
- [x] tests/test_compute.py -- 51 tests covering compute_pwm and compute_local_pwm

### Test suite (2026-04-03)
- [x] tests/test_pssm.py -- 48 tests
- [x] tests/test_utils.py -- 32 tests
- [x] tests/test_kmers.py -- 30 tests
- [x] tests/test_compute.py -- 51 tests
- [x] tests/test_regression.py -- 30 tests
- [x] tests/test_visualization.py -- 36 tests
- [x] tests/test_genomic.py -- 31 tests
- Total: 258+ tests, all passing

## Next Steps
- [ ] Performance: Optimize `init_energies()` inner loop with Numba or vectorised NumPy (currently pure Python)
- [ ] Implement `motif_enrichment()` -- database screening
- [ ] Load standard motif databases (JASPAR, HOMER) from bundled data files
- [ ] GPU-differentiable variant of the regression optimizer (PyTorch backend)
