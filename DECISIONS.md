# Architecture Decisions

## 1. NumPy-based computation, not PyTorch/GPU

**Decision**: Use NumPy for all core computation.

**Rationale**: The primary goal of pyprego is to faithfully mirror the R prego
package behaviour on CPU. The R package uses Rcpp (C++ via R) and does not
require GPU hardware. A NumPy backend ensures:

- Identical numerical behaviour to the R package (both use 64-bit floats by default)
- No CUDA/GPU dependency -- runs anywhere Python + NumPy are available
- Clean array interfaces: all hot-path functions accept/return `np.ndarray`, so
  swapping to torch tensors later is a matter of changing the array backend, not
  the algorithm

The existing `pyrego` package already explored the GPU path with PyTorch. pyprego
deliberately takes the opposite approach to serve users who need CPU-only
reproducibility.

## 2. pandas DataFrames for PSSM and spatial models

**Decision**: Represent PSSMs as `pd.DataFrame(pos, A, C, G, T)` and spatial
models as `pd.DataFrame(bin, spat_factor)`.

**Rationale**: The R package returns tibbles with these exact column names.
Using DataFrames:

- Makes the Python objects directly inspectable (`print(pssm)` is informative)
- Allows column-name based access (`pssm["A"]`) matching R semantics
- Follows the pymisha convention of using pandas for tabular genomic data
- Provides easy serialisation to CSV / parquet / YAML

Helper functions `pssm_to_array()` and `pssm_dataframe()` bridge between the
DataFrame representation and raw `(L, 4)` NumPy arrays for computation.

## 3. Dataclass for RegressionResult, not a dict

**Decision**: Use `@dataclass` for the regression output instead of returning a
plain dict (as the R package does with a named list).

**Rationale**: A dataclass gives:

- IDE autocompletion and type checking
- A `predict()` method directly on the result object
- Explicit field documentation
- Easy serialisation via `to_dict()`

The R package returns a list; Python has better options.

## 4. Optional pymisha dependency for genomic functions

**Decision**: The `genomic` module imports pymisha only at call time, not at
module import time.

**Rationale**: Most pyprego users will work with plain sequence strings, not
genomic databases. Making pymisha a hard dependency would block installation
for users who don't need it. The lazy-import pattern (with a clear error
message) follows pymisha's own conventions for optional dependencies.

## 5. Flat module structure (no sub-packages)

**Decision**: Single-level `pyprego/` package with one file per domain area.

**Rationale**: The R prego package has ~30 R source files but no sub-directory
structure. pyrego went with nested sub-packages (`pyrego.backends`,
`pyrego.regression`, etc.) which added complexity. For a package of this size,
a flat module layout is simpler and easier to navigate. If modules grow large,
they can be promoted to sub-packages later without breaking the public API
(since everything is re-exported from `__init__.py`).

## 6. Placeholder pattern for unimplemented functions

**Decision**: Functions that are not yet implemented raise `NotImplementedError`
with a descriptive message, but have complete type-annotated signatures and
docstrings.

**Rationale**: This lets us:

- Lock in the API surface early
- Generate correct documentation / stubs
- Write tests against the expected interface
- Implement incrementally without changing signatures

## 7. Testing follows pymisha conventions

**Decision**: pytest with conftest.py fixtures, markers for slow/genomic tests.

**Rationale**: Consistency with the lab's other Python packages.

## 8. Pure Python regression optimizer (no C++ extension initially)

**Decision**: Port the C++ `PWMLRegression` coordinate descent optimizer to pure
Python/NumPy rather than wrapping the C++ code with pybind11.

**Rationale**:
- Faster development cycle — no compilation step needed
- Easier debugging and modification
- The algorithm is I/O-bound on sequence scanning, not compute-bound on linear algebra
- Pure Python inner loop is slow (~30s for 200 seqs × 280bp × 12mer) but acceptable for development
- Future optimization paths: Numba JIT for the hot loop, or GPU-differentiable PyTorch version
- C++ extension can be added later as a drop-in replacement for the inner loop

## 9. Cross-validation via rank-averaging (matching C++ exactly)

**Decision**: When using multiple folds, rank all (position, move) combinations
per-fold, then average ranks, and pick the best average rank — matching the
C++ integer division behavior exactly.

**Rationale**: This reproduces the C++ behavior faithfully. Some implementations
use score-averaging, but rank-averaging is more robust to scale differences
between folds and matches the published algorithm.

## 10. Model export as JSON (not RDS)

**Decision**: Export regression models as JSON files rather than Python pickles
or R RDS files.

**Rationale**:
- JSON is human-readable and language-agnostic
- No need for R-Python model interop (user requirement: "doesn't have to read R models")
- YAML export also supported for complex nested structures
- Avoids pickle security concerns

## 11. MotifDB as a Python class (not S4)

**Decision**: Port R's S4 `MotifDB` class to a regular Python class with
`__getitem__`, `__len__`, `__iter__` dunder methods.

**Rationale**: Python classes provide the same encapsulation as S4 without
metaclass complexity. The subscript operator (`db["GATA"]`) is more Pythonic
than R's S4 slot access.

## 12. Three-tier testing strategy

**Decision**: Three categories of tests:
1. **Golden master tests**: Generate reference data from R, compare Python output
2. **Ported R tests**: Translate R testthat tests to pytest
3. **Pure Python tests**: Additional edge cases, properties, integration tests

**Rationale**: Golden master tests verify numerical compatibility with R.
Ported R tests ensure feature parity. Pure Python tests cover edge cases
and integration scenarios that R tests don't.

## 13. GPU-differentiable design considerations

**Decision**: Keep clean array interfaces (functions accept/return NumPy arrays)
and separate the optimization loop from score computation.

**Rationale**: The user plans a future GPU/differentiable version. By keeping
`compute_pwm` as a pure function of (sequences, pssm, spat_factors) → scores,
it can be replaced with a PyTorch version that supports autograd. The
`_PWMLRegression` class separates the coordinate descent logic (which would
be replaced by gradient descent) from the energy computation (which stays
the same).

## 14. iceqream compatibility

**Decision**: Ensure all functions used by iceqream R package are implemented:
`intervals_to_seq`, `regress_pwm`, `compute_pwm`, `compute_local_pwm`,
`extract_pwm`, `create_motif_db`, `trim_pssm`, `pssm_rc`, `pssm_match`,
`bits_per_pos`, `all_motif_datasets`, `plot_pssm_logo`, `plot_spat_model`.

**Rationale**: pyprego is a prerequisite for a Python version of iceqream.
These functions were identified by analyzing iceqream's R source code for
`prego::` calls.
