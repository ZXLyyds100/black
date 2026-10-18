# Black #2334: overloaded operators and string processing

This directory contains genuine before/after output from the Python formatter, displayed
in a small **CLI-output viewer, not a Black product UI**. The screenshots and recording
were captured in Chromium at the same 1440 × 950 viewport. There are no model calls,
API-key-dependent services, or mocked formatter results.

## Conditions and reproduction

Pristine baseline: `psf/black` main at `11cff1312bf96e02bce9a467d0b5a2dd65300642`.

After: the proposed implementation of issue #2334, based on that exact revision. The
screenshots describe the proposed source patch, not a released Black version. Both runs
use the same macOS ARM64 host and CPython 3.12.14 interpreter. The shallow development
checkout reports a generated development version.

Run in each checkout:

```sh
python -m black --preview --enable-unstable-feature string_processing \
  --line-length 20 -c 'x /= "directory subdirectory file.txt"'
```

Before:

```python
x /= "directory subdirectory file.txt"
```

After:

```python
x /= (
    "directory"
    " subdirectory"
    " file.txt"
)
```

Replace `/=` with `/`, `**`, or `@=` for the other recorded examples. The
`runtime-comparison.json` file contains the complete changed-output matrix.
`viewer.html` can be opened locally to inspect the same saved output.

## Verification

- Focused transformer and formatting suite: 315 passed.
- Full `tox -e py -- -rs`: 629 passed and 1 optional-IPython skip before installing
  Jupyter extras, followed by 82 Jupyter tests passed.
- `tox -e run_self`: passed.
- Strict documentation build, `sphinx-build -a -b html -W docs/ docs/_build/`: passed.
- Full repository-configured pre-commit: all hooks passed; no checks were skipped or
  dependency versions overridden.
- 47 inputs × 4 modes × 2 revisions = 376 real CLI calls. Stable and ordinary preview
  outputs are byte-identical on this bounded matrix.
- An actual Python class implements all 26 arithmetic/augmented overload methods.
  Executing the formatted program on both revisions preserves all observed string
  values.
- An additional 514 generated syntax cases per revision are checked using Black's
  non-fast AST-equivalence and idempotence checks. Stable/ordinary preview output is
  unchanged, with no newly introduced string-processing failures.
- 68 tracked Python source files, using identical pristine inputs in both revisions, are
  also byte-identical in stable and ordinary preview modes.

The expanded syntax checks caught and helped fix a `%` associativity regression before
publication. Six permanent precedence regression cases protect it. One unrelated
bare-starred-tuple AST-safety failure is reproduced on both pristine main and patched
code; it is not counted as a passing supplemental case.

The host exports `NO_COLOR=1`. It was unset **only for test subprocesses** because the
repository's color tests otherwise also fail on pristine main. No global setting,
product code, dependency pin, or test skip was used to hide that failure.

## Limits

Only macOS ARM64 / CPython 3.12.14 has been verified locally. Other Python versions,
Linux, Windows, mypyc wheels, fuzzing, and the complete hosted diff-shades project
corpus have not been run locally. Hosted CI and maintainer review are separate gates and
are not inferred from local test success.

The implementation and evidence are AI-assisted. The two self-review passes and
automated checks are not represented as independent human review.
