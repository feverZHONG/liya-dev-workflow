---
tier: T2  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

## Recipe 1: Local breakpoint

Easiest. Edit the file:

```python
def compute(x, y):
    result = some_helper(x)
    breakpoint()           # <-- drops into pdb here
    return result + y
```

Run the code normally. You land at the `breakpoint()` line with full access to locals.

**Don't forget to remove `breakpoint()` before committing.** Use `git diff` or a pre-commit grep:
```bash
rg -n 'breakpoint\(\)' --type py
```

## Recipe 2: Launch a script under pdb (no source edits)

```bash
python -m pdb path/to/script.py arg1 arg2
# Lands at first line of script
(Pdb) b path/to/script.py:42
(Pdb) c
```

## Recipe 3: Debug a pytest test

The hermes test runner and pytest both support this:

```bash
# Drop to pdb on failure (or on any raised exception):
scripts/run_tests.sh tests/path/to/test_file.py::test_name --pdb

# Drop to pdb at the START of the test:
scripts/run_tests.sh tests/path/to/test_file.py::test_name --trace

# Show locals in tracebacks without pdb:
scripts/run_tests.sh tests/path/to/test_file.py --showlocals --tb=long
```

Note: `scripts/run_tests.sh` uses xdist (`-n 4`) by default, and pdb does NOT work under xdist. Add `-p no:xdist` or run a single test with `-n 0`:

```bash
scripts/run_tests.sh tests/foo_test.py::test_bar --pdb -p no:xdist
# or
source .venv/bin/activate
python -m pytest tests/foo_test.py::test_bar --pdb
```

This bypasses the hermetic-env guarantees — fine for debugging, but re-run under the wrapper to confirm before pushing.

