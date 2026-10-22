# gs1-product
Example GS1 Product Repository that uses Zensical

## Python environment

Use Python 3.14 (see `.python-version`) and the repository's `.venv`:

```bash
python3.14 -m venv .venv
.venv/bin/python -m pip install -r requirements-docs.txt
.venv/bin/python -m pip check
.venv/bin/zensical build --clean --strict
```

`requirements-docs.txt` pins Zensical, its dependencies and pip. CI uses the
same Python minor version and dependency pins. This repository has no local
Python package to install in editable mode and no Python test suite.

To upgrade the documentation environment:

```bash
.venv/bin/python -m pip install --upgrade --force-reinstall zensical
.venv/bin/python -m pip install --upgrade pip click deepmerge Jinja2 Markdown MarkupSafe pathspec Pygments pymdown-extensions PyYAML tomli
.venv/bin/python -m pip check
.venv/bin/python -m pip list --outdated
.venv/bin/zensical build --clean --strict
```

After a successful upgrade, refresh the pins in `requirements-docs.txt` from
`.venv/bin/python -m pip freeze --all`. Review the changes before committing.
